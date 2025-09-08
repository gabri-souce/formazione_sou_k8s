FROM ubuntu:22.04

ARG SSH_PUB_KEY
ARG SSH_PASSWORD

RUN apt-get update && apt-get install -y openssh-server sudo && \
    mkdir -p /var/run/sshd && \
    useradd -m -s /bin/bash gabriele && \
    echo "gabriele ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers && \
    echo "gabriele:${SSH_PASSWORD}" | chpasswd

RUN mkdir -p /home/gabriele/.ssh && \
    echo "${SSH_PUB_KEY}" > /home/gabriele/.ssh/authorized_keys && \
    chmod 700 /home/gabriele/.ssh && \
    chmod 600 /home/gabriele/.ssh/authorized_keys && \
    chown -R gabriele:gabriele /home/gabriele/.ssh

RUN sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin no/' /etc/ssh/sshd_config && \
    sed -i 's/#PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config

EXPOSE 22
CMD ["/usr/sbin/sshd","-D"]




