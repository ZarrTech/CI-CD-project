FROM jenkins/jenkins:lts
LABEL project="vprofile"

# Install Maven and JDK
USER root
RUN apt-get update && apt-get install -y maven default-jdk

# set the user back to the jenkins user
USER jenkins