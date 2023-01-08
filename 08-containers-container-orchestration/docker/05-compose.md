# Docker Compose

In the beginning was Fig. Fig was a powerful tool, created by a company called Orchard, and it was the best way to manage multi-container Docker apps. It was a Python tool that sat on top of Docker, and let you define entire multi-container apps in a single YAML file. You could then deploy and manage the lifecycle of the app with the fig command-line tool.

Behind the scenes, Fig would read the YAML file and use Docker to deploy and manage the app via the Docker API. It was a good thing.

In fact, it was so good, that Docker, Inc. acquired Orchard and re-branded Fig as Docker Compose. The commandline tool was renamed from fig to docker-compose, and continues to be an external tool that gets bolted on top of the Docker Engine. Even though it’s never been fully integrated into the Docker Engine, it’s always been popular and widely used.

As things stand today, Compose is still an external Python binary that you have to install on a Docker host. You define multi-container (microservices) apps in a YAML file, pass the YAML file to the docker-compose command line, and Compose deploys it via the Docker API. However, April 2020 saw the announcement of the Compose Specification. This is aimed at creating an open standard for defining multi-container cloud-native apps. The ultimate aim being to greatly simplify the code-to-cloud process.

The specification will be community-led and separate from the docker-compose implementation from Docker, Inc. This helps maintain becker governance and clearer lines of demarcation. However, we should expect Docker to implement the fill spec in docker-compose.