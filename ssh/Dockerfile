FROM centos:8

ARG user=root

USER ${user}

#Install Java (Optional)
RUN yum install -y java-1.8.0-openjdk-devel.x86_64
ENV JAVA_HOME /usr/lib/jvm/java


# Install a basic SSH server
RUN yum install openssh-server openssh-clients -y && \
    sed -i 's|session    required     pam_loginuid.so|session    optional     pam_loginuid.so|g' /etc/pam.d/sshd && \
    mkdir -p /var/run/sshd && \

# Set password for the jenkins user (you may want to alter this).
    echo "root:root" | chpasswd

# Standard SSH port
EXPOSE 22
RUN env | grep _ >> /etc/environment
RUN ssh-keygen -A
CMD ["/usr/sbin/sshd", "-D"]
