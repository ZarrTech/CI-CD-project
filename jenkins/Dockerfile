FROM jenkins/jenkins:lts
LABEL project="vprofile"

# Switch to root user for installing dependencies
USER root

#install maven docke jdk
RUN apt-get update && apt-get install -y maven docker.io openjdk-17-jdk

# Set back to Jenkins user
USER jenkins
