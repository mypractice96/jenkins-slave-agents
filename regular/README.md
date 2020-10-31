Method 1 : Attach Docker Container    (Can be user only for docker cloud) 

Prerequisites:
Docker image must have Java installed.
Docker image CMD must either be empty or simply sit and wait forever, e.g. /bin/bash.
The Jenkins remote agent code will be copied into the container and then run using the Java that's installed in the container.

Sample Image : Build this dockerfile
