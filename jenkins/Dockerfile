FROM jenkins/jenkins:lts
LABEL project="vprofile"

# Switch to root user for installing dependencies
USER root

# Install Maven, Docker, and OpenJDK 17
RUN apt-get update && apt-get install -y \
    maven \
    docker.io \
    openjdk-17-jdk \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

RUN curl -SL https://github.com/docker/compose/releases/download/v2.32.4/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose \
    && chmod +x /usr/local/bin/docker-compose

# Install AWS CLI
RUN apt-get update && \
    apt-get install -y awscli

# Add Jenkins user to the Docker group
RUN usermod -aG docker jenkins

# Switch back to Jenkins user
USER jenkins
