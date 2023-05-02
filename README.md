# App Manifests

This repository is one of a set providing a demonstration of a [GitOps](https://www.weave.works/technologies/gitops/) approach using [Argo CD](https://argo-cd.readthedocs.io/en/stable/) as the deployment tool and Kustomise for defining the Kubernetes manifests.
The full set of repositories in the demo are:
- [ops-manifests](https://github.com/rw-gitops-demo/ops-manifests) - the Kubernetes ops manifests, including Argo CD
- [app-manifests](https://github.com/rw-gitops-demo/app-manifests) - the Kubernetes application manifests
- [apples-service](https://github.com/rw-gitops-demo/apples-service) - an example Node.js application
- [gitops-scripts](https://github.com/rw-gitops-demo/gitops-scripts) - a set of scripts installed into the manifests repos as a git submodule

Please follow the [ops-manifests](https://github.com/rw-gitops-demo/ops-manifests) README before continuing with the steps outlined here.

The purpose of this repository is to manage the microservices running in the Kubernetes cluster in each environment.
This demo currently only defines a single `apples-service` as an example, but other microservices could also be defined here.

## Setting up the demo

This repo has a single GitHub action workflow called `CD`, which is designed to be triggered by workflows from other repositories in order to deploy new application releases. The trigger itself is parameterised with the name of the service and the release tag.
The [apples-service](https://github.com/rw-gitops-demo/apples-service) repository provides an example of this setup.

For other repositories to trigger the `CD` action, [GitHub requires that this be done using a Personal Access Token](https://docs.github.com/en/actions/using-workflows/triggering-a-workflow#triggering-a-workflow-from-a-workflow).
Create a new fine-grained personal access token for your GitHub user with your GitHub organisation as the resource owner and scoped to access this repository only.
This token requires Read and Write access to actions for this repository.

Next, in your organisation's settings, add a new organisation secret named `DEPLOY_PAT` for the token you just created.
This can be scoped to all public repositories.
Microservice repos will now be able to reference this secret in their workflows.

Finally, the workflow requires that `dev` and `prod` environments are configured for the repository.
Add a protection rule to the `prod` environment to require approval for deployments to production.

### Cloning this repository

This repo has a git submodule pointing to the [gitops-scripts](https://github.com/rw-gitops-demo/gitops-scripts) repo.
To clone the repo and pull the files in the module at the same time, execute:
```shell
git clone --recurse-submodules git@github.com:rw-gitops-demo/app-manifests.git
```
You will then need to run the following to set up the git hooks:
```shell
npx husky install
```

## Running the demo

The [ops-manifests](https://github.com/rw-gitops-demo/ops-manifests) README outlines how to set up a Kubernetes cluster with Argo CD to deploy all the applications defined in this repository.
Once the cluster is running, use port forwarding to access the Apples Service at http://127.0.0.1:8081.
```shell
kubectl port-forward --namespace=apps svc/apples-service 8081:3000
```

## Configuration changes

The GitOps workflow as showcased here separates the application configuration from its code. This allows configuration changes such as number of replicas and environment variables to be updated in the cluster without the need to rebuild the application. The changes can be made by simply commtting them to the repository. Argo CD will automatically sync them to the cluster.

This repository uses the "environment-per-folder" approach as described in [this blog post](https://codefresh.io/blog/how-to-model-your-gitops-environments-and-promote-releases-between-them/). The blog post describes the advantages of this approach over the often used "environment-per-branch" approach as well as considerations for carefully propagating changes through each environment.
