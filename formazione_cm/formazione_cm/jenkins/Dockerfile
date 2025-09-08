FROM jenkins/jenkins:lts

USER root

# Install dependencies
RUN apt-get update && \
    apt-get install -y docker.io podman ansible sudo ssh && \
    usermod -aG docker jenkins

# Optional: install ansible collections or plugins if needed
USER jenkins

EXPOSE 8080
