# jenkins-slave-agents

When we need scalable jenkins, then we use to run the jobs in the dynamic containers.


Using Docker cloud plugins, we can connect to the slave containers in the following ways.







********************************************************************************************************************
Method 1 : Attach Docker Container    (Can be user only for docker cloud) 

Prerequisites:
Docker image must have Java installed.
Docker image CMD must either be empty or simply sit and wait forever, e.g. /bin/bash.
The Jenkins remote agent code will be copied into the container and then run using the Java that's installed in the container.

Sample Image : Build this dockerfile

__________ Dockerfile__________

FROM centos:8
RUN yum install -y java-1.8.0-openjdk-devel.x86_64
ENV JAVA_HOME /usr/lib/jvm/java
CMD ["/bin/bash"]
______________________________


*********************************************************************************************************************














*********************************************************************************************************************

Method 2 : JNLP (can be used for both docker cloud as well as Kubernetes)

Prerequisites:
Jenkins master has to be accessible over network from the container.
Docker image must have Java installed.
Docker image must launch agent.jar by itself or using the EntryPoint Arguments below.


__________ Dockerfile__________
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
___________________________________________



__________________________jenkins-agent_________________________________________
#!/usr/bin/env sh

# The MIT License
#
#  Copyright (c) 2015-2020, CloudBees, Inc.
#
#  Permission is hereby granted, free of charge, to any person obtaining a copy
#  of this software and associated documentation files (the "Software"), to deal
#  in the Software without restriction, including without limitation the rights
#  to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
#  copies of the Software, and to permit persons to whom the Software is
#  furnished to do so, subject to the following conditions:
#
#  The above copyright notice and this permission notice shall be included in
#  all copies or substantial portions of the Software.
#
#  THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
#  IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
#  FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
#  AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
#  LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
#  OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
#  THE SOFTWARE.

# Usage jenkins-agent.sh [options] -url http://jenkins [SECRET] [AGENT_NAME]
# Optional environment variables :
# * JENKINS_TUNNEL : HOST:PORT for a tunnel to route TCP traffic to jenkins host, when jenkins can't be directly accessed over network
# * JENKINS_URL : alternate jenkins URL
# * JENKINS_SECRET : agent secret, if not set as an argument
# * JENKINS_AGENT_NAME : agent name, if not set as an argument
# * JENKINS_AGENT_WORKDIR : agent work directory, if not set by optional parameter -workDir
# * JENKINS_WEB_SOCKET: true if the connection should be made via WebSocket rather than TCP
# * JENKINS_DIRECT_CONNECTION: Connect directly to this TCP agent port, skipping the HTTP(S) connection parameter download.
#                              Value: "<HOST>:<PORT>"
# * JENKINS_INSTANCE_IDENTITY: The base64 encoded InstanceIdentity byte array of the Jenkins master. When this is set,
#                              the agent skips connecting to an HTTP(S) port for connection info.
# * JENKINS_PROTOCOLS:         Specify the remoting protocols to attempt when instanceIdentity is provided.

if [ $# -eq 1 ]; then

	# if `docker run` only has one arguments, we assume user is running alternate command like `bash` to inspect the image
	exec "$@"

else

	# if -tunnel is not provided, try env vars
	case "$@" in
		*"-tunnel "*) ;;
		*)
		if [ ! -z "$JENKINS_TUNNEL" ]; then
			TUNNEL="-tunnel $JENKINS_TUNNEL"
		fi ;;
	esac

	# if -workDir is not provided, try env vars
	if [ ! -z "$JENKINS_AGENT_WORKDIR" ]; then
		case "$@" in
			*"-workDir"*) echo "Warning: Work directory is defined twice in command-line arguments and the environment variable" ;;
			*)
			WORKDIR="-workDir $JENKINS_AGENT_WORKDIR" ;;
		esac
	fi

	if [ -n "$JENKINS_URL" ]; then
		URL="-url $JENKINS_URL"
	fi

	if [ -n "$JENKINS_NAME" ]; then
		JENKINS_AGENT_NAME="$JENKINS_NAME"
	fi

	if [ "$JENKINS_WEB_SOCKET" = true ]; then
		WEB_SOCKET=-webSocket
	fi

	if [ -n "$JENKINS_PROTOCOLS" ]; then
		PROTOCOLS="-protocols $JENKINS_PROTOCOLS"
	fi

	if [ -n "$JENKINS_DIRECT_CONNECTION" ]; then
		DIRECT="-direct $JENKINS_DIRECT_CONNECTION"
	fi

	if [ -n "$JENKINS_INSTANCE_IDENTITY" ]; then
		INSTANCE_IDENTITY="-instanceIdentity $JENKINS_INSTANCE_IDENTITY"
	fi

	# if java home is defined, use it
	JAVA_BIN="java"
	if [ "$JAVA_HOME" ]; then
		JAVA_BIN="$JAVA_HOME/bin/java"
	fi

	# if both required options are defined, do not pass the parameters
	OPT_JENKINS_SECRET=""
	if [ -n "$JENKINS_SECRET" ]; then
		case "$@" in
			*"${JENKINS_SECRET}"*) echo "Warning: SECRET is defined twice in command-line arguments and the environment variable" ;;
			*)
			OPT_JENKINS_SECRET="${JENKINS_SECRET}" ;;
		esac
	fi
	
	OPT_JENKINS_AGENT_NAME=""
	if [ -n "$JENKINS_AGENT_NAME" ]; then
		case "$@" in
			*"${JENKINS_AGENT_NAME}"*) echo "Warning: AGENT_NAME is defined twice in command-line arguments and the environment variable" ;;
			*)
			OPT_JENKINS_AGENT_NAME="${JENKINS_AGENT_NAME}" ;;
		esac
	fi

	#TODO: Handle the case when the command-line and Environment variable contain different values.
	#It is fine it blows up for now since it should lead to an error anyway.

	exec $JAVA_BIN $JAVA_OPTS -cp /usr/share/jenkins/agent.jar hudson.remoting.jnlp.Main -headless $TUNNEL $URL $WORKDIR $WEB_SOCKET $DIRECT $PROTOCOLS $INSTANCE_IDENTITY $OPT_JENKINS_SECRET $OPT_JENKINS_AGENT_NAME "$@"
fi

________________________________________________________________________________________________________


***********************************************************************************************************************************************************************









*********************************************************************************************************************
Method 3 : SSH 

Prerequisites:
The docker container's mapped SSH port, typically a port on the docker host, has to be accessible over network from the master.
Docker image must have sshd installed.
Docker image must have Java installed.
Log in details configured as per ssh-slaves plugin.

_________________Dockerfile______________________
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
__________________________________________________________


*********************************************************************************************************************
