# Java Web Project Example

A simple java web project, managed by Apache Maven. The contained Dockerfile defines a multi-stage build process to create an image for our application. Copies the pom.xml & src files separately to make efficient use of image layer cachine.

```docker
docker build -t java-web-project .
docker run -d -p 8080:8080 java-web-project
```