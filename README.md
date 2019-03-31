## Installing NGINX Ingress Controller + Signal Sciences next-gen WAF (Using Signal Sciences NGINX Native Module)

### Introduction:

This repository contains an example of embedding the Signal Sciences Agent in the [ingress-nginx](https://github.com/kubernetes/ingress-nginx) Ingress controller:

- [/sigsci-module-nginx-ingress/Dockerfile](/sigsci-module-nginx-ingress/Dockerfile)
  - This container is a "wrapper" for the default `nginx-ingress-controller` container and serves to add the Signal Sciences nginx Module files. You can set the image version in the Dockerfile, version used here is 0.22.0
- [/sigsci-agent/Dockerfile](/sigsci-agent/Dockerfile)
  - This container is a sidecar, running the Signal Sciences Agent, that will be included in the same pod as the nginx-ingress-controller
- [mandatory.yaml](mandatory.yaml)
  - This file contains a modified template of the Generic Ingress Controller Deployment as described at https://kubernetes.github.io/ingress-nginx/deploy/#prerequisite-generic-deployment-command. The main additions are:
    - Changing the ingress container to load the custom Signal Sciences Module/ingress container and adding Volume mounts for socket file communication between the Module/ingress container and Agent sidecar container:
    ```
    containers:
      - name: nginx-ingress-controller
        image: myregistry/sigsci-module-nginx-ingress:0.22.0
        ...
        volumeMounts:
          - name: sigsci-socket
            mountPath: /var/run/
    ```
    - Loading the Signal Sciences Module in nginx.conf via ConfigMap:
    ```
    kind: ConfigMap
    apiVersion: v1
    data:
      main-snippet: load_module /usr/lib/nginx/modules/ngx_http_sigsci_module.so;
    metadata:
      name: nginx-configuration
    ```
    - Adding a container for the Signal Sciences Agent:
    ```
      - name: sigsci-agent
        image: myregistry/sigsci-agent:3.21.0
        volumeMounts:
          - name: sigsci-socket
            mountPath: /var/run/
        env:
          - name: SIGSCI_ACCESSKEYID
            value: SETME
          - name: SIGSCI_SECRETACCESSKEY
            value: SETMETOO
    ```
    - And defining the volume used above:
    ```
    volumes:
      - name: sigsci-socket
        emptyDir: {}
    ```

### Setup:

#### 1. Set Agent Keys in mandatory.yaml

#### 2. Build the nginx ingress + Signal Sciences Module container 
*Set whatever registry + repository name you'd like here, just be sure to set* `controller.image.repository:` *to match in [mandatory.yaml](mandatory.yaml)*
```
docker build -t myregistry/sigsci-module-nginx-ingress:0.22.0 sigsci-module-nginx-ingress/
```

#### 3. Build the Signal Sciences Agent sidecar container
*Again, set whatever registry + repository name you'd like here, just be sure to set* `controller.extraContainers.image:` *to match in [mandatory.yaml](mandatory.yaml)*
```
docker build -t myregistry/sigsci-agent:3.21.0 sigsci-agent/
```

#### 4. Deploy using modified Generic Deployment
```
kubectl apply -f mandatory.yaml
```

#### 5. Create service to expose Ingress Controller

Steps depend on cloud provider:
https://kubernetes.github.io/ingress-nginx/deploy/#provider-specific-steps