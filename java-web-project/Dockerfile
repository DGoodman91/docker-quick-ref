FROM maven AS build
WORKDIR ./
COPY . .
RUN mvn package

FROM tomcat:9.0
COPY --from=build /target/java-web-project.war /usr/local/tomcat/webapps/ROOT.war