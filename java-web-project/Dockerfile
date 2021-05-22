FROM maven AS build
WORKDIR ./
COPY pom.xml ./
RUN mvn package
COPY . .

FROM tomcat:9.0
COPY --from=build /target/java-web-project.war /usr/local/tomcat/webapps/ROOT.war