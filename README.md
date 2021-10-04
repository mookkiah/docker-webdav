# NGINX WebDAV container

## Build
```bash
docker build . -t mookkiah/docker-webdav
```

## Usage:

Without external volume
```bash
docker run --detach --name webdav --publish 7000:8080 mookkiah/docker-webdav
```

With external(host) volume
```bash
docker run --restart always --detach --name webdav --publish 7000:8080 \
           --env UID=$UID --volume $PWD:/media mookkiah/docker-webdav
```

## Securing WebDav
Optionally you can add two environment variables to require HTTP basic authentication:

* WEBDAV_USERNAME
* WEBDAV_PASSWORD

### Example:

```bash
docker run --restart always --detach --name webdav --publish 7000:8080 \
           --env WEBDAV_USERNAME=webdav --env WEBDAV_PASSWORD=S0m37h1n6C0mp13x \
           --env UID=$UID --volume $PWD:/media mookkiah/docker-webdav
```

## Kubernetes Deployment and Services:

Store below file in file webdav.yaml and apply using `kubectl apply -f webdav.yaml`
```yaml
# kubectl create deployment nginx-webdav --image=mookkiah/webdav --dry-run -oyaml

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
      - image: mookkiah/docker-webdav
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

To check access
```
curl http://localhost:7000
```
with authentication
```
curl -u webdav:S0m37h1n6C0mp13x http://localhost:7000
```

### To create a folder
```
curl -X MKCOL -u webdav:S0m37h1n6C0mp13x http://localhost:7000/home/
```
### To upload a file
```
curl -u webdav:S0m37h1n6C0mp13x -T webdav.yaml http://localhost:7000/webdav.yaml
```
### To delete a folder
```
curl -X DELETE -u webdav:S0m37h1n6C0mp13x http://localhost:7000/home/webdav.yaml
```
### To delete a file 
```
curl -X DELETE -u webdav:S0m37h1n6C0mp13x http://localhost:7000/home
```

## Cleanup

Docker
```bash
docker stop webdav
docker rm webdav
```

Kubernetes
```
kubernetes delete -f webdav.yaml
```

## Note:
If you get internal server error while making changes via webdav
Try execute below command to confirm if this is permission issue.
```
kubectl exec -it <pod name>  -- chmod +777 /media -R
`` 