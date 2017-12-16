---
layout: post
title: 'Optimizing ASP.NET Core Docker images'
published: true
---

With ASP.NET Core 2.0, Microsoft included a new NuGet Package to manage your projects' dependencies on the ASP.NET Core framework, the [Microsoft.AspNetCore.All](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/metapackage) metapackage.

The advantages of using this package instead of individually referencing the packages you need are twofold:
- You save up time looking for specific minor versions of the packages. Referencing AspNetCore.All 2.0.0 and upgrading to 2.0.3 is going to ensure you get all the bugfixes without having to worry about looking for individual package updates.

- You save up space in published applications. This requires, however, an installation on the target of the ASP .NET Core Runtime Store - an installable manifest-defined directory that contains the relevant packages, and that can be installed in deployment targets for your application.

With this last point, especially, the advantages are clear for Docker users. If you look at the definition of the [official Microsoft ASP.NET Core docker images](https://github.com/aspnet/aspnet-docker/blob/master/2.0/jessie/runtime/Dockerfile), you can see that the runtime store is being setup. What does this mean? Well, it means that if you have 1, 5, or 300 different docker images based off the official microsoft/aspnetcore image, you will only download these packages once - any other layers beyond that, defined in your own Dockerfile, will not be redownloaded.

This makes for huge savings in space (and, directly, time). In my quick experiment, the savings were astoundingly high - a relatively simple microservice with its only NuGet dependencies being Newtonsoft.Json, MediatR and AspNetCore libraries went from 54 MB to 6. 

### Things I'm looking into next

- Using the [Microsoft.Packaging.Tools.Trimming](https://www.nuget.org/packages/Microsoft.Packaging.Tools.Trimming/1.1.0-preview1-25818-01) library to cut some more size off. It sounds [really good](https://github.com/dotnet/standard/blob/release/2.0.0/Microsoft.Packaging.Tools.Trimming/docs/trimming.md), but the fact that it's still in pre-release worries me a bit. We'll see.

- Creating my own base image that inherits from aspnetcore and fills the runtime store with some more packages that may be common to all my images (Newtonsoft.Json is always going to be there, and that's another few megabytes we can save!). This would probably have to have a dedicated build step before the rest of my build, to ensure I keep up with the minor updates Microsoft publishes to its aspnetcore version.

### References / Read More

- https://andrewlock.net/the-microsoft-aspnetcore-all-metapackage-is-huge-and-thats-awesome-thanks-to-the-net-core-runtime-store-2/