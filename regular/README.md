## Method 1 : Attach Docker Container (Can be user only for docker cloud) 

### Prerequisites:
1) Docker image must have Java installed.
2) Docker image CMD must either be empty or simply sit and wait forever, e.g. /bin/bash.
3) The Jenkins remote agent code will be copied into the container and then run using the Java that's installed in the container.

**For sample Image : Build the above dockerfile**
