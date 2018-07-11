SQL performance & query tuning
A standard noSQL database - MongoDB
Building basic CI/CD pipeline
Web & service hosting in cloud, basics - AWS
Basic web performance - minification, sprites, CDN, caching etc.

a docker image is like a bunch of instructions, dependencies to run the app
container is a process that runs on a docker host
container is live version of the image (like and instantiation of the class)

instead of setting up the whole server (again n again), all you need is a server that can run docker
images can be versioned, tagged, swapped in and out
to scale, simply spin up more containers from the image (will run as separate processes)

a dockerfile is a plain text, that says how to build the image (name: "Dockerfile", in same directory as Program.cs)

------------------------------------------
Sample Dockerfile for .NET Core app
------------------------------------------
FROM microsoft/dotnet:latest
COPY . /app
WORKDIR /app

RUN dotnet restore
RUN dotnet build

EXPOSE 5000/tcp
ENV ASPNETCORE_URLS http://*:5000

ENTRYPOINT dotnet run
------------------------------------------

```bash
> docker build -t MyAppImageName /buildImageFromDir #create an image
> docker images #see all images
> docker run -it -p 5000:5000 MyAppImageName #run the image as container 
# -d for background -it to show console logs 
# -p 5000:5000 to expose post 5000 from docker to 5000 in machine
> docker ps # show status/monitor running containers
> docker ps --format 'table {{.Names}}\t{{.Image}}\t{{.Status}}\t{{.Ports}}' # formatted
> docker stop container_name

> docker save -o ImageFilename.tar MyAppImageName
```
