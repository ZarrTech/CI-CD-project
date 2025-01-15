FROM jenkins/jenkins:lts
LABEL project="vprofile"

# Install Maven and JDK
USER root

# Update apt-get and install Maven and OpenJDK 11
RUN apt-get update && \
    apt-get install -y software-properties-common && \
    add-apt-repository ppa:openjdk-r/ppa && \
    apt-get update && \
    apt-get install -y maven openjdk-11-jdk

# set the user back to the jenkins user
USER jenkins