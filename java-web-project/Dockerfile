FROM maven AS build
WORKDIR /build
COPY pom.xml /build/
RUN mvn dependency:resolve --fail-never
COPY src /build/src
RUN mvn package

FROM tomcat:9.0
COPY --from=build /target/java-web-project.war /usr/local/tomcat/webapps/ROOT.war