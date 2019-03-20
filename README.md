<img align="right" src="https://raw.githubusercontent.com/startxfr/docker-images/master/travis/logo-small.svg?sanitize=true">

# docker-images-example-ooconv


Example of a LibreOffice document converter using the startx s2i builder [startx/sv-ooconv](https://hub.docker.com/r/startx/sv-ooconv). 
For more information on how to use this image, **[read startx ooconv image guideline](https://github.com/startxfr/docker-images/blob/master/Services/ooconv/README.md)**.

## Running this example in OKD (aka Openshift)

### Create a sample application

```bash
# Create a openshift project
oc new-project startx-example-ooconv
# start a new application (build and run)
oc process -f https://raw.githubusercontent.com/startxfr/docker-images/master/Services/ooconv/openshift-template-build.yml -p APP_NAME=myapp | oc create -f -
# Watch when resources are available
sleep 30 && oc get all
```

### Create a personalized application

- **Initialize** a project
  ```bash
  export MYAPP=myapp
  oc new-project ${MYAPP}
  ```
- **Add template** to the project service catalog
  ```bash
  oc create -f https://raw.githubusercontent.com/startxfr/docker-images/master/Services/ooconv/openshift-template-build.yml -n startx-example-ooconv
  ```
- **Generate** your current application definition
  ```bash
  export MYVERSION=0.1
  oc process -n startx-example-ooconv -f startx-ooconv-build-template \
      -p APP_NAME=v${MYVERSION} \
      -p APP_STAGE=example \
      -p BUILDER_TAG=latest \
      -p SOURCE_GIT=https://github.com/startxfr/docker-images-example-ooconv.git \
      -p SOURCE_BRANCH=master \
      -p MEMORY_LIMIT=256Mi \
  > ./${MYAPP}.definitions.yml
  ```
- **Review** your resources definition stored in `./${MYAPP}.definitions.yml`
- **build and run** your application
  ```bash
  oc create -f ./${MYAPP}.definitions.yml -n startx-example-ooconv
  sleep 15 && oc get all
  ```
- **Test** your application
  ```bash
  oc describe route -n startx-example-ooconv
  curl http://<url-route>
  ```

## Running this example with source-to-image (aka s2i)

### Create a sample application

```bash
# Build the application
s2i build https://github.com/startxfr/docker-images-example-ooconv startx/sv-ooconv startx-ooconv-sample
# Run the application
docker run --rm -d -p 8777:2002 startx-ooconv-sample
# Test the sample application
telnet -h localhost 8777
```

### Create a personalized application

- **Initialize** a project directory
  ```bash
  git clone https://github.com/startxfr/docker-images-example-ooconv.git ooconv-myapp
  cd ooconv-myapp
  rm -rf .git
  ```
- **Develop** and create a personalized page
  ```bash
  cat << "EOF"
  <html><head></head><body><h1>My Web Application</h1></body></html>
  EOF > test.html
  ```
- **Build** your current application with startx ooconv builder
  ```bash
  s2i build . startx/sv-ooconv:latest startx-ooconv-myapp
  ```
- **Run** your application and test it
  ```bash
  docker run --rm -d -p 8777:2002 startx-ooconv-myapp
  ```
- **Test** your application
  ```bash
  telnet -h localhost 8777
  ```
