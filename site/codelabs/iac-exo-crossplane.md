summary: Instantiate a Crossplane managed K8s cluster on Exoscale
id: cloud-aws-vpn-services
categories: Kubernetes, Crossplane, Exoscale
tags: introduction, fhb-pe
status: Published
authors: Thomas Schuetz

# Create GitHub Repositories with Crossplane

## Introduction
In this lab, we will demonstrate how to provision GitHub repositories using Crossplane. Crossplane is an open-source Kubernetes add-on that enables you to manage cloud resources using Kubernetes. It extends the Kubernetes API to allow you to create, configure, and manage cloud resources using Kubernetes manifests. Crossplane is a CNCF sandbox project and is part of the Cloud Native Computing Foundation.

### Big Picture

### Prerequisites
- A Kubernetes Cluster
- Solid understanding of the Kubernetes basics
- Shell access to the Kubernetes Cluster
- Bash/Zsh shell (WSL is also possible)

### Objectives
- Install a Crossplane managed GitHub Repository on Exoscale

## Step 1: Verify the Kubernetes Cluster
If you have not installed a Kubernetes Cluster yet, you can follow the instructions in the [Create a Kubernetes Cluster with IaC](./iac-exo-k8s-cluster) lab.

After you have successfully installed the Kubernetes Cluster, you can verify the installation by running the following command:

```bash
kubectl get nodes
```

If you see the nodes of the cluster, you have successfully installed the Kubernetes Cluster.

## Step 2: Install Crossplane
If you haven't installed Helm yet, please install it by following the instructions in the Helm documentation (https://helm.sh/docs/intro/install/).

To install Crossplane, you can use the following Helm command:

```bash
helm repo add crossplane-stable https://charts.crossplane.io/stable
helm repo update
helm install crossplane --namespace crossplane-system crossplane-stable/crossplane --create-namespace --wait
```

After the installation, you can verify the installation by running the following command:

```bash
kubectl get pods -n crossplane-system
```

If you see the pods of Crossplane, you have successfully installed Crossplane.

## Step 3: Install the Exoscale Provider
Crossplane Providers are extensions to Crossplane that enable it to manage a new kind of resource in an external system. They are essentially plugins that are installed into a Crossplane cluster to add new functionality. For example, a provider could add support for managing databases in a specific cloud provider, or it could add support for managing DNS records in a DNS system. Each provider brings along a set of Custom Resource Definitions (CRDs) that define the resources it can manage. Once a provider is installed, users can create instances of these CRDs to provision and manage resources in the external system.

As we want to manage resources in Exoscale, we need to install the Exoscale Provider.

To install the GitHub Provider, you have to create the following YAML file and apply it to the Kubernetes Cluster (as described [here](https://marketplace.upbound.io/providers/coopnorge/provider-github/v0.10.0)):

```yaml
apiVersion: pkg.crossplane.io/v1
kind: Provider
metadata:
   name: provider-github
spec:
   package: xpkg.upbound.io/coopnorge/provider-github:v0.10.0
```

After you have created the file, you can apply it to the Kubernetes Cluster by running the following command:

```bash
kubectl apply -f &lt;filename&gt;
```

You should see the provider being installed by running the following command:

```bash
kubectl get providers.pkg.crossplane.io
```

To create resources in GitHub, you need to create a secret with your GitHub credentials. Therefore, you have to obtain a GitHub Personal Access Token. You can create a new token by accessing GitHub and going to Settings -> Developer settings -> Personal access tokens -> Generate new token. Make sure to select the `repo` scope.

After you have obtained the token, you can create a secret by creating the following YAML file and applying it to the Kubernetes Cluster ([docs](https://github.com/coopnorge/provider-github/blob/main/README.md)).

```yaml
---
apiVersion: v1
kind: Secret
metadata:
  name: provider-secret
  namespace: upbound-system
type: Opaque
stringData:
  credentials: "{\"token\":\"${GH_TOKEN}\",\"owner\":\"${GH_OWNER}\"}"

---
apiVersion: github.upbound.io/v1beta1
kind: ProviderConfig
metadata:
  name: default
spec:
  credentials:
    source: Secret
    secretRef:
      name: provider-secret
      namespace: crossplane-system
      key: credentials
```

After you have created the file, you can apply it to the Kubernetes Cluster by running the following command:

```bash
kubectl apply -f &lt;filename&gt;
```

## Step 4: Create a GitHub Repository with Crossplane
To create a GitHub Repository, you can create the following YAML file and apply it to the Kubernetes Cluster:

```yaml
---
apiVersion: repo.github.upbound.io/v1alpha1
kind: Repository
metadata:
  name: my-crossplane-repo
spec:
  providerConfigRef:
    name: default
  forProvider:
    name: my-crossplane-repo
    autoInit: true
    visibility: private
---
apiVersion: repo.github.upbound.io/v1alpha1
kind: Branch
metadata:
  name: add-file-branch
spec:
  forProvider:
    repositoryRef:
      name: my-crossplane-repo
    branch: add-file-branch
  providerConfigRef:
    name: default
---
apiVersion: repo.github.upbound.io/v1alpha1
kind: RepositoryFile
metadata:
  name: sample-file-dot-txt
spec:
  forProvider:
    file: sample-file.txt
    content: |
      I am an crossplane
      provider. This should be nice.
    commitMessage: "managed by crossplane provider"
    commitAuthor: "Crossplane Github Provider"
    commitEmail: "github-provider@crossplane.com"
    branch: add-file-branch
    repositoryRef:
      name: my-crossplane-repo
  providerConfigRef:
    name: default
```

After you have created the file, you can apply it to the Kubernetes Cluster by running the following command:

```bash
kubectl apply -f &lt;filename&gt;
```

### Verify the GitHub Repository
To verify the GitHub Repository, you can access GitHub and check if the repository has been created and there is a file in there.

## Conclusion
In this lab, we have demonstrated how to provision GitHub repositories using Crossplane. Crossplane is a powerful tool that allows you to manage cloud resources using Kubernetes. It extends the Kubernetes API to allow you to create, configure, and manage cloud resources using Kubernetes manifests. 












