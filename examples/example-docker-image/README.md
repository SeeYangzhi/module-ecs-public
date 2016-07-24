**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-ecs>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-ecs/blob/master/examples/example-docker-image/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# Example Docker Image

This folder defines a small Docker image we can use for testing. It contains a Node.js webapp that returns the text
in the environment variable `SERVER_TEXT`, or "Hello world!' if that environment variable is not set.

This image has been pushed to the gruntwork Docker Hub account under the name `gruntwork/docker-test-webapp`.

## Build the image

```
docker build -t gruntwork/docker-test-webapp .
```

## Run the image

```
docker run --rm -it -p 3000:3000 gruntwork/docker-test-webapp
```
