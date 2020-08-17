# Setting up Orchard Core Web App with MS SQL Server running in Docker Containers via Docker Compose

## Orchard Core in Docker

## [YouTube Video](https://youtu.be/sy7c3EKEtYc)

[![OrchardSkillsYouTubeThumbNailDocker](https://user-images.githubusercontent.com/59172485/90452425-53e88c00-e0ab-11ea-842d-f00f77f55fc3.png)](https://youtu.be/sy7c3EKEtYc)

# Introduction

Streamline your development workflow by using Docker to stand up and run SQL Server instances quickly and without fuss.

## What is Docker?

In short however, Docker allows you to “spin-up” instances of applications, (e.g. SQL Server, Redis, etc.), really quickly without having to go through laborious, (and often confusing), installations. So from a developer perspective – it’s awesome – you can concentrate on coding, and not get side-tracking on installing an instance of SQL Server on Linux, (for example).

## Why Would You Use it?

- **Portability.** As containers are self-***contained*** they can run on any platform that runs Docker, making them easy to stand up and run on a wide variety of platforms.
- **Scalability.** With the use of additional “orchestration” you can spin up multiple container instances to support increased load.
- **Performance.** Containers generally perform better than their VM counterparts.
- **Ubiquity.** The level of Docker adoption in industry means that it’s a great skill to have.

## Why SQL Server?

I picked SQL Server as the target app for this article for 2 reasons:

1. I like and use SQL Server all the time because I'm familiar with it.
2. It’s traditionally a “Windows-only” system, so proving you can spin up an instance on any platform running Docker is awesome.

## Run SQL Server In Docker

Now, (assuming you’ve installed Docker), type the following at the command line, (don’t worry I’ll take you through everything below):

Let's go through a docker run with SQL.

### -e ‘ACCEPT_EULA=Y’

The “-e” flag is essentially an “Environment Flag”, which allows us to, (again not surprisingly), configure the Container Environment. In this particular case, we are specifying that we accept the End User Licensing Agreement, (EULA), for SQL Server by passing in a ‘Y’ value.

#### -e ‘SA_PASSWORD=Pa$$w0rd2020’

Another Environment Flag, this time we are setting up the Server Administrator, (SA), account for SQL Server. The SA account is, (as the name suggests), the local SQL Server Admin account – so be careful!

GOTCHA! You’ll need to provide a SA password that adheres to the SA Password Policy, otherwise the SQL Server instance will fail to run, (you’ll get a GUID returned, as the Container does run briefly, but will subsequently stop if a weak password is provided!).

### -e ‘MSSQL_PID=Express’

Our final Environment flag, this one specifies the “flavour” of SQL Server, in this instance we’re passing in ‘Express’ as we only require the free version.

### -p 1433:1433

This is our “Port Mapping” flag, it maps the Containers “internal” port to an externally facing port on our local machine. Without this, we could not connect into our SQL Server Container instance. Here we are mapping the internal port 1433 to an external port of 1433, (note we could choose any unused “external” port).

### -d mcr.microsoft.com/mssql/server:2017-latest-ubuntu

The ‘-d’ flag tells Docker to run our Container in “detached mode”, so it kind of runs in the “background”, therefore we’ll get a prompt back at our command line.

The last part of this command is just the name of the image we want followed by the version we want, (image name and version are separated by a colon ‘:’).

Note: You’ll see that we are using the Ubuntu Linux image of SQL Server, (there are of course Windows versions available). I choose this version as Linux and Mac PC’s running Docker can only use “Linux Containers”. Only Windows PC’s can run both Windows and Linux Containers, so this was the more ubiquitous choice.


![Orchard-Core-Docker-01](https://user-images.githubusercontent.com/59172485/90451525-45997080-e0a9-11ea-9cf8-e9523ee5f1c3.png)

Launch VSCode and load the solution.

![Orchard-Core-Docker-02](https://user-images.githubusercontent.com/59172485/90451527-46320700-e0a9-11ea-9dec-c470e5507932.png)

Select the extensions icon on the left and Install the Docker extension.

![Orchard-Core-Docker-03](https://user-images.githubusercontent.com/59172485/90451530-46ca9d80-e0a9-11ea-81f1-4f8d6d9f6bc2.png)

## Docker Clean

To start out with a fresh clean docker environment run the Docker Clean commands:

```
docker stop $(docker ps -a -q)
docker rm $(docker ps -a -q)
docker rmi $(docker images -q)
```

![Orchard-Core-Docker-04](https://user-images.githubusercontent.com/59172485/90451531-47633400-e0a9-11ea-9919-34f6b23e6478.png)

To run Docker Compose right click on the Docker Compose YML file and select "Compose Up"

## Docker File

```
#See https://aka.ms/containerfastmode to understand how Visual Studio uses this Dockerfile to build your images for faster debugging.

FROM mcr.microsoft.com/dotnet/core/aspnet:3.0-buster-slim AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443

FROM mcr.microsoft.com/dotnet/core/sdk:3.0-buster AS build
WORKDIR /src
COPY ["MyOrchardCoreCMS.csproj", ""]
COPY ["NuGet.config", ""]
RUN dotnet restore "./MyOrchardCoreCMS.csproj"
COPY . .
WORKDIR "/src/."
RUN dotnet build "MyOrchardCoreCMS.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "MyOrchardCoreCMS.csproj" -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "MyOrchardCoreCMS.dll"]
```


## Docker Compose YML File

```
version: "3.7"

services:
  myorchardcorecms:
    build: .
    ports:
        - "8080:80"    
    depends_on:
        - mssql

  mssql:
    image: mcr.microsoft.com/mssql/server:2017-latest-ubuntu
    environment:
        ACCEPT_EULA: "Y"
        SA_PASSWORD: "Pa55w0rd2020"
        MSSQL_PID: Express
    ports:
        - "1433:1433"
```


## Docker Compose Up Shell Script

```
#!/bin/bash
docker-compose up
```


## Docker Compose Down Shell Script

```
#!/bin/bash
docker-compose stop
```


![Orchard-Core-Docker-05](https://user-images.githubusercontent.com/59172485/90451532-47633400-e0a9-11ea-904f-0dfe198fd387.png)

With your favorite browser, browse to localhost:8080.

![Orchard-Core-Docker-06](https://user-images.githubusercontent.com/59172485/90451533-47fbca80-e0a9-11ea-9c48-bc105c406a87.png)

Enter your site name, Select Blog for the recipe, select Sql Server for the database, enter the connection string credentials and then press the Finish Setup button.

![Orchard-Core-Docker-08](https://user-images.githubusercontent.com/59172485/90451537-48946100-e0a9-11ea-81e1-f4d94677f9a4.png)

Log into the Admin Dashboard by browsing to: localhost:8080/admin.

![Orchard-Core-Docker-09](https://user-images.githubusercontent.com/59172485/90451540-492cf780-e0a9-11ea-9103-9acf6b016685.png)

Enter your credentials and press the "Log in" button.

![Orchard-Core-Docker-10](https://user-images.githubusercontent.com/59172485/90451542-492cf780-e0a9-11ea-9fa5-4ca0a9ed9eba.png)

Now you have logged into the Admin Dashboard, You are ready to set started with Orchard Core CMS.

# Conclusion

As a developer, I don’t want to waste time standing up the infrastructure to support my coding endeavors – I just want to code! So for me Docker is an excellent tool I use to streamline my development workflow.

# GitHub

The complete source code is located [here](https://github.com/OrchardSkills/OrchardSkills.OrchardCore.Docker).