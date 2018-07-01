---
layout: post
title: 'SignalR in .NET Core - File Downloads'
published: true
---
This is just a small example of how to use the new [SignalR](https://github.com/aspnet/signalr) version distributed with .NET Core from 2.1 onwards in order to perform a file download between a client and a server. 

The usage scenario here was a server with a Web API/MVC logic that hosts a SignalR hub, combined with a series of clients that exist in machines distributed across the world that are connected to that hub. It should be possible for the hub to trigger a file download on those clients, sending them a zip, or a JPG, or whatever.

This sample is of the **server-side** code:
```csharp
public ChannelReader<byte[]> DownloadFileStream(string path)
{
    Guard.Against<FileNotFoundException>(!File.Exists(path), $"No file found at {path}");
    var channel = Channel.CreateUnbounded<byte[]>();
    _ = WriteToChannel(channel, path);
    return channel.Reader;
    async Task WriteToChannel(ChannelWriter<byte[]> writer, string filePath)
    {
        using (FileStream fs = File.OpenRead(filePath))
        {
            byte[] buffer = new byte[8 * 1024];
            int len;
            while ((len = await fs.ReadAsync(buffer, 0, buffer.Length)) > 0)
            {
                await writer.WriteAsync(buffer);
                await Task.Delay(1000);
            }
            writer.Complete();
        }
    }
}
```

And on the **client-side**:
```csharp
private async Task BeginStreamingFile(string path)
{
    string fileName = Path.GetFileName(path);
    var channel = await _connection.StreamAsChannelAsync<byte[]>("DownloadFileStream", path, CancellationToken.None);
    while (await channel.WaitToReadAsync())
    {
        using (FileStream fs = File.OpenWrite(fileName)) { 
            while (channel.TryRead(out byte[] buffer))
            {
                await fs.WriteAsync(buffer, 0, buffer.Length);
            }
        }
    }
}
```

The solution I had to begin the streaming was for the client to get a message with the path of the file it wanted to download from a helper method in a controller I'd do a HTTP POST on. Before I was able to develop something cleverer, we decided that, for the purposes of the feature we were designing, we could just use a feature we already had designed and do the download of a ZIP file from the API with what we needed. Hopefully in the future we'll come back to this!

### References / Read More

- https://radu-matei.com/blog/signalr-core/ - Please check out Radu Matei's post on ASP.NET SignalR. It was really helpful in getting us started with it, especially as we began this work before a stable version of .NET Core 2.1 was even out, so the documentation didn't cover a lot of what we needed.