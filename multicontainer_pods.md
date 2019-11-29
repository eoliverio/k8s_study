# MULTI-CONTAINER PODS

Multi-container pods, despite having multiple design patters, are created in the same way in the Pod definition YAML file.

### Sample YAML definition
Simply add another container in the container list under spec.
```
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
  labels:
    name: simple-webapp
spec:
  containers:
  - name: simple-webapp
    image: simple-webapp
    ports:
    - containerPort: 8080
  - name: log-agent
    image: log-agent
```

#### Using `kubectl exec` command to get a shell to a running container in a multi-container pod
```
kubectl exec -it <pod-name> -c <container-name> bash
```

## Design Patterns

Information below is taken from the transcript of `Section 4 Multi-Container PODs` in the `Kubernetes Certified Application Developer (CKAD) with Tests` course by Mumshad Mannambeth

### Sidecar

A good example of a Sidecar pattern is deploying a logging agent alongside a web server to collect logs and forward them to a central log server.

### Adapter

Say we have multiple applications generating logs in different formats.

It will be hard to process the various formats on the central logging server.

So before sending the logs to the central server we would like to convert the logs to a common format.

For this, we deploy an Adapter container. The Adapter container processes the logs before sending it to the central server.

### Ambassador

So your application communicates to different database instances at different stages of development.

A local database for development, one for testing and another for production.

You must make sure to modify this connectivity in your application code depending on the environment you are deploying your application to.

You may choose to outsource such logic to a separate container within your pod so that your application can always refer to a database at local host and the new container will proxy that request to the right database.

This is known as an Ambassador container.
