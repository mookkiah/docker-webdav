# NGINX WebDAV container

Build:
```bash
docker build . -t mookkiah/webdav
```

Usage:

Without external volume
```bash
docker run --restart always --detach --name webdav --publish 7000:8080 \
           --env UID=$UID  mookkiah/webdav
```

Without external(host) volume
```bash
docker run --restart always --detach --name webdav --publish 7000:8080 \
           --env UID=$UID --volume $PWD:/media mookkiah/webdav
```

Optionally you can add two environment variables to require HTTP basic authentication:

* WEBDAV_USERNAME
* WEBDAV_PASSWORD

Example:

```bash
docker run --restart always --detach --name webdav --publish 7000:8080 \
           --env WEBDAV_USERNAME=myuser --env WEBDAV_PASSWORD=mypassword \
           --env UID=$UID --volume $PWD:/media mookkiah/webdav
```

Kubernetes Deployment and Services:

```

# kubectl create deployment nginx-webdav --image=ionelmc/webdav --dry-run -oyaml

apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-webdav
  name: nginx-webdav
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-webdav
  template:
    metadata:
      labels:
        app: nginx-webdav
    spec:
      containers:
      # Forcing specific version of image as it is from public and untrusted. 
      - image: mookkiah/webdav
        name: webdav
        env:
          - name: WEBDAV_USERNAME
            value: "webdav"
          - name: WEBDAV_PASSWORD
            value: "S0m37h1n6C0mp13x" #SomethingComplex
          - name: UID
            value: "0"  #String formatted UID of the file system directory /media (expect internal server error due to permission issue if not set right value)
---

# kubectl expose deployment nginx-webdav --type=LoadBalancer --name=webdav-service --port=8080 --dry-run -oyaml 
apiVersion: v1
kind: Service
metadata:
  labels:
    app: nginx-webdav
  name: webdav-service
spec:
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: nginx-webdav
  type: LoadBalancer

```