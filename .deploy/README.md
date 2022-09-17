# skaffold
Manifest for handling services in the Kubernetes cluster. 


## Table of Contents
- [Configuration](#configuration)
- [Adding new Deployments](#adding-new-deployments)
- [Adding a new Environment](#adding-a-new-environment)


## Configuration
The configuration for each namespace can be found in the folder environment. The environments correspond to the deployment environments to the specifc project. In each environemnt folder there is a kustomization file. This file is responsible for tying together all configurations and resources.

### Prerequisites
This setup requires Kubernetes, Skaffold and Nginx Ingress Controller, see [Installation in README](../README.md)

### Ingress
To learn more about Ingress traffic handling see [Nginx Ingress Controller](https://kubernetes.github.io/ingress-nginx/user-guide/basic-usage/).

## Adding new Deployments
When adding a new deployment/app/service there are several steps that needs to be taken. Consider the following steps for adding a new service to the local cluster deployment. Look at existing files in each step for inspiration.

**Add the Base**
- Add a Base Deployment
  - Add the deployment, e.g. `.skaffold/base/k8s/deployments/`*`new-service.yaml`*.
  - Include the file as a resource in the kustomization `.skaffold/base/k8s/deployments/kustomization.yaml`.

- Add a Base Service 
  
  If the new service needs to connect to other apps in the cluster you need to add a service which exposes the app.

  - Add the service, e.g. `.skaffold/base/k8s/services/`*`new-service.yaml`*.
  - Include the file as a resource in the kustomization `.skaffold/base/k8s/services/kustomization.yaml`.

**Add a Dockerfile** 

- Create a Dockerfile in the `.skaffold/local` directory.

**Add an Ingress Route**
- Add a route to the new service to `.skaffold/local/k8s/ingress.yaml`. Make sure to use the port and service name that was specified in `.skaffold/base/k8s/services/`*`new-service.yaml`*.

**Add Skaffold Configuration**
- Add a link to the Dockerfile in `skaffold.yaml` under the *local profile*. Make sure the image matches the container image specified in `.skaffold/base/k8s/deployments/`*`new-service.yaml`*.

```yaml
// skaffold.yaml

...
profiles:
- name: local
  build:
    artifacts:
    - image: nss-image-new-service
      docker:
        dockerfile: .skaffold/local/Dockerfile.new-service
...
```

## Adding a new Environment

When adding a new deployable environment there are a few steps included, the easiest method is to copy the `.skaffold/local` directory and add/remove what is needed. 

Different environments should be as similar to each other as possible to avoid unexpected production issues but may for instance alter in what *.env*-variables they use or other performance measures that needs to be put in place for specific environments. 

### Steps for adding an environment

For this example we are creating a new dev environment.

**Create the Docker- and k8s-files**
- Copy the `.skaffold/local` environment into a new directory `.skaffold/dev`. 
- Remove and add files not relevant to the dev environment, e.g., *mongodb* as that is only used for local development. 

**Update the Namespace**
- Update the namespace name from `nss-dummy-local` to `nss-dummy-dev` in `.skaffold/dev/namespace.yaml`.

**Update Kustomize**
- In `.skaffold/dev/kustomize.yaml`
  - Update the namespace to the new namespace name `nss-dummy-dev` 
  - Update the commonLabels environment from `local` to `dev`.

**Add Skaffold Configuration**
- Add a new profile to `skaffold.yaml`

```yaml
// skaffold.yaml

...
profiles:
- name: dev
  build:
    artifacts:
    - image: nss-image-new-service
      docker:
        dockerfile: .skaffold/dev/Dockerfile.new-service
...
```