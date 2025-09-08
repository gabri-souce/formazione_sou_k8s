FROM alpine:3.18

ARG SSH_PUB_KEY
ARG SSH_PASSWORD

# Install SSH server, bash e sudo
RUN apk add --no-cache openssh bash sudo

# Create user 'gabriele' con password e sudo senza richiesta password
RUN adduser -D -s /bin/bash gabriele && \
    echo "gabriele ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers && \
    echo "gabriele:${SSH_PASSWORD}" | chpasswd

# Configura SSH key authentication
RUN mkdir -p /home/gabriele/.ssh && \
    echo "${SSH_PUB_KEY}" > /home/gabriele/.ssh/authorized_keys && \
    chmod 700 /home/gabriele/.ssh && \
    chmod 600 /home/gabriele/.ssh/authorized_keys && \
    chown -R gabriele:gabriele /home/gabriele/.ssh

# Abilita SSHD
EXPOSE 22
CMD ["/usr/sbin/sshd","-D"]





