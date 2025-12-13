FROM ubuntu:latest

# install the dependencies
RUN apt-get update -y && \
    apt-get install python3 python3-pip openssh-server sudo -y && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

# create the ansible user & allow password logins
RUN useradd -m -s /bin/bash -G sudo ansible && \
    echo "ansible:p@ssw0rd" | chpasswd && \
    mkdir -p /var/run/sshd

# allow password based sudo
RUN echo "ansible ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/ansible

# expose the ssh port
EXPOSE 22

# run SSH as a daemon
CMD ["/usr/sbin/sshd", "-D"]