FROM tomcat:10-jdk21
LABEL project="vprofile"
LABEL name="zarrcloud"

RUN rm -rf /usr/local/tomcat/webapps/*
COPY target/vprofile.war /usr/local/tomcat/webapps/ROOT.war

EXPOSE 8080
CMD ["catalina.sh", "run"]
VOLUME /usr/local/tomcat/webapps