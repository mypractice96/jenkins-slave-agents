## Method 2 : JNLP (can be used for both docker cloud as well as Kubernetes)

### Prerequisites:
1) Jenkins master has to be accessible over network from the container.
2) Docker image must have Java installed.
3) Docker image must launch agent.jar by itself or using the EntryPoint Arguments below.

**For sample image, build the above Dockerfile**
