# How to Deploy Private docker registry

## Getting Started
If you need to store your docker image in private docker registry you can deploy registry image from hub.docker.com to create private registry\
`docker run --name private-registry -p 5000:5000 registry:2`\
The registry default running in Port 5000