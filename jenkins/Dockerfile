FROM jenkins/jenkins:lts
LABEL project="vprofile"

# Switch to root user for installing dependencies
USER root

# Install Maven and OpenJDK 11
RUN apt-get update && \
    apt-get install -y wget gnupg software-properties-common && \
    wget -qO - https://packages.adoptium.net/artifactory/api/gpg/key/public | apt-key add - && \
    echo "deb https://packages.adoptium.net/artifactory/deb focal main" > /etc/apt/sources.list.d/adoptium.list && \
    apt-get update && \
    apt-get install -y maven temurin-11-jdk && \
    apt-get clean

# Set JAVA_HOME environment variable
ENV JAVA_HOME=/usr/lib/jvm/temurin-11-jdk-amd64

# Set default Java to JDK 11
RUN update-alternatives --install /usr/bin/java java $JAVA_HOME/bin/java 1 && \
    update-alternatives --set java $JAVA_HOME/bin/java

# Set back to Jenkins user
USER jenkins
