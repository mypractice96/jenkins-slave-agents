FROM centos:8

#Install Java
RUN yum install -y java-1.8.0-openjdk-devel.x86_64
ENV JAVA_HOME /usr/lib/jvm/java

#Download agent.jar from https://repo.jenkins-ci.org/public/org/jenkins-ci/main/remoting/
ARG VERSION=4.5
ARG user=root
ARG AGENT_WORKDIR=${user}/agent
RUN curl --create-dirs -fsSLo /usr/share/jenkins/agent.jar https://repo.jenkins-ci.org/public/org/jenkins-ci/main/remoting/${VERSION}/remoting-${VERSION}.jar
RUN chmod 755 /usr/share/jenkins \
     && chmod 644 /usr/share/jenkins/agent.jar \
      && ln -sf /usr/share/jenkins/agent.jar /usr/share/jenkins/slave.jar
USER ${user}
ENV AGENT_WORKDIR=${AGENT_WORKDIR}
RUN mkdir ${user}/.jenkins && mkdir -p ${AGENT_WORKDIR}

VOLUME ${user}/.jenkins
VOLUME ${AGENT_WORKDIR}
WORKDIR ${user}

#Now run the jenkins-agent script (which has  {java -jar agent.jar} command)
COPY jenkins-agent /usr/local/bin/jenkins-agent
RUN chmod +x /usr/local/bin/jenkins-agent &&\
    ln -s /usr/local/bin/jenkins-agent /usr/local/bin/jenkins-slave

ENTRYPOINT ["jenkins-agent"]
