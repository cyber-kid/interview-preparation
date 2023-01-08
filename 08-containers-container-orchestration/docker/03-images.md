# Docker Images

* [Images and layers](#images-and-layers)
* [Sharing image layers](#sharing-image-layers)
* [Multi-architecture images](#multi-architecture-images)
* [Moving to production with Multi-stage Builds](#moving-to-production-with-multi-stage-builds)

A Docker image is a unit of packaging that contains everything required for an application to run. This includes; application code, application dependencies, and OS constructs. If you have an application’s Docker image, the only other thing you need to run that application is a computer running Docker.

Images are made up of multiple layers that are stacked on top of each other and represented as a single object. Inside of the image is a cut-down operating system (OS) and all of the files and dependencies required to run an application.

In fact, you can stop a container and create a new image from it. With this in mind, images are considered build-time constructs, whereas containers are run-time constructs.

### Images and layers
A Docker image is just a bunch of loosely-connected read-only layers, with each layer comprising one or more files.

All Docker images start with a base layer, and as changes are made and new content is added, new layers are added on top.

Docker employs a storage driver that is responsible for stacking layers and presenting them as a single unified filesystem/image.

### Sharing image layers
Multiple images can, and do, share layers. This leads to efficiencies in space and performance.

### Multi-architecture images
One of the best things about Docker is its simplicity. However, as technologies grow, things get more complex. This happened for Docker when it started supporting multiple different platforms and architectures such as Windows and Linux, on variations of ARM, x64, PowerPC, and s390x. All of a sudden, popular images had versions for different platforms and architectures. As developers and operators, we had to make sure we were pulling the correct version for the platform and architecture we were using. This broke the smooth Docker experience.

Fortunately, Docker and Docker Hub have a slick way of supporting multi-arch images. This means a single image, such as golang:latest, can have an image for Linux on x64, Linux on PowerPC, Windows x64, Linux on different versions of ARM, and more. To be clear, we’re talking about a single image tag supporting multiple platforms and architectures. But it means you can run a simple docker image pull goloang:latest from any platform or architecture and Docker will pull the correct image for your platform and architecture.

### Moving to production with Multi-stage Builds
Multi-stage builds have a single Dockerfile containing multiple FROM instructions. Each FROM instruction is a new build stage that can easily COPY artefacts from previous stages.

```
FROM node:latest AS storefront
WORKDIR /usr/src/atsea/app/react-app
COPY react-app .
RUN npm install
RUN npm run build

FROM maven:latest AS appserver
WORKDIR /usr/src/atsea
COPY pom.xml .
RUN mvn -B -f pom.xml -s /usr/share/maven/ref/settings-docker.xml dependency:resolve
COPY . .
RUN mvn -B -s /usr/share/maven/ref/settings-docker.xml package -DskipTests

FROM java:8-jdk-alpine AS production
RUN adduser -Dh /home/gordon gordon
WORKDIR /static
COPY --from=storefront /usr/src/atsea/app/react-app/build/ .
WORKDIR /app
COPY --from=appserver /usr/src/atsea/target/AtSea-0.0.1-SNAPSHOT.jar .
ENTRYPOINT ["java", "-jar", "/app/AtSea-0.0.1-SNAPSHOT.jar"]
CMD ["--spring.profiles.active=postgres"]
```

The first thing to note is that the Dockerfile has three FROM instructions. Each of these constitutes a distinct build stage. Internally, they’re numbered from the top starting at 0. However, we’ve also given each stage a friendly name.

An important thing to note, is that COPY --from instructions are used to only copy production-related application code from the images built by the previous stages. They do not copy build artefacts that are not needed for production.

It’s also important to note that we only need a single Dockerfile, and no extra arguments are needed for the docker image build command!