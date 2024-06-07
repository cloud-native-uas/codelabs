summary: Automate DNS, Certificates and Secrets in Kubernetes
id: k8s-infra-services
categories: Kubernetes
tags: advanced, fhb-pe
status: Published
authors: Thomas Schuetz

# Automate DNS, Certificates and Secrets in Kubernetes

## Introduction
In this lab, you will learn how we can deal with DNS, Certificates and Secrets in a Cloud Native way. We will use the ExternalDNS, cert-manager and SealedSecrets tools to automate the management of DNS records, certificates and secrets in a Kubernetes cluster.

### Prerequisites
- A Kubernetes that is publicly accessible (https://github.com/cloud-native-uas/example-code/tree/main/sks-cluster)
- A domain that you can manage (provided at UAS)
- Helm installed (https://helm.sh/docs/intro/install/)

### Objectives
* You will install and configure ExternalDNS to automatically manage DNS records outside of Kubernetes
* You will install and configure cert-manager to automatically manage certificates in Kubernetes
* You will install and configure SealedSecrets to automatically manage secrets in Kubernetes

## Step 1: Prepare your Workspace
* Ensure that you are using a proper IDE and have access to a terminal
* Ensure that you have access to a Kubernetes cluster
* Create a directory for this lab and navigate into it
* Also ensure that `kubectl` is installed and configured to access your Kubernetes cluster
* You should be able to run `kubectl` commands without specifying the `--kubeconfig` flag

## Step 2: Install an Ingress Controller
To make our services accessible from the internet, we need an Ingress Controller. We will use the NGINX Ingress Controller for this lab.

### Install the NGINX Ingress Controller
To install the NGINX Ingress Controller, you can use the following command:

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install nginx-ingress ingress-nginx/ingress-nginx --create-namespace --namespace nginx-ingress --wait
```

### Verify the Installation
After the installation, you can verify the installation by running the following command:

```bash
kubectl get pods -n nginx-ingress
```

If you see the pods of the NGINX Ingress Controller, you have successfully installed the NGINX Ingress Controller and can proceed with the next steps.

## Step 3: Install Cert-Manager
Cert-Manager is a Kubernetes add-on to automate the management and issuance of TLS certificates from various issuing sources. It will ensure certificates are valid and up to date.

The add-on uses Issuers (Namespaced) or ClusterIssuers (Cluster-wide) to manage the issuance of certificates. In this lab, we will use a ClusterIssuer to issue certificates. The Issuer is responsible for communicating with the certificate authority to obtain certificates.

### Install Cert-Manager
To install Cert-Manager, you can use the following command:

```bash
helm repo add jetstack https://charts.jetstack.io
helm repo update
helm install cert-manager jetstack/cert-manager --create-namespace --namespace cert-manager --set installCRDs=true --wait
```

### Add the ACME Staging ClusterIssuer
To issue certificates, we need to add a ClusterIssuer. In this lab, we will use the Let's Encrypt staging environment to issue certificates. This is useful for testing purposes.

You can add the ClusterIssuer with the following manifest:

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-staging
spec:
  acme:
    # You must replace this email address with your own.
    # Let's Encrypt will use this to contact you about expiring
    # certificates, and issues related to your account.
    email: user@example.com
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      # Secret resource that will be used to store the account's private key.
      name: example-issuer-account-key
    # Add a single challenge solver, HTTP01 using nginx
    solvers:
      - http01:
          ingress:
            ingressClassName: nginx
```

### Verify the Installation
After the installation, you can verify the installation by running the following command:

```bash
kubectl get pods -n cert-manager
```

Furthermore, you can verify the installation of the ClusterIssuer by running the following command:

```bash
kubectl get clusterissuers.cert-manager.io
```

If you see the pods of Cert-Manager, you have successfully installed Cert-Manager and can proceed with the next steps.

## Step 4: Install ExternalDNS
ExternalDNS is a Kubernetes controller that configures DNS resources to match the current state of Kubernetes resources. It makes our applications accessible via public DNS names and ensures that our cert-manager endpoints are reachable too.

### Install ExternalDNS
To install ExternalDNS, you can use the following command:

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm install external-dns bitnami/external-dns --create-namespace --namespace external-dns --set provider=digitalocean --set digitalocean.secretName=external-dns-password --wait
```

Furthermore, you need to create a secret with your DigitalOcean API key. You can do this by running the following command:

```bash
kubectl create secret generic external-dns-password --from-literal=digitalocean_api_token=YOUR_DIGITAL_OCEAN_API_TOKEN -n external-dns
```

### Verify the Installation
After the installation, you can verify the installation by running the following command:

```bash
kubectl get pods -n external-dns
```

If you see the pods of ExternalDNS, you have successfully installed ExternalDNS and can proceed with the next steps.

## Step 5: Install SealedSecrets
When you are working with Kubernetes Secrets and use them in manifests, you can provide them as plain text or base64 encoded. When using version control systems, this might be a security risk, as the secrets are stored in plain text in the repository. Therefore, you can use mechanisms like Sealed Secrets or various key management systems to encrypt the secrets. In this task, you will create a Sealed Secret and use it in the application.

### Install Sealed Secrets
Sealed Secrets is implemented as a Kubernetes Controller. Therefore, you have to install it to your cluster. This can be done with the following command:
* `kubectl apply -f https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.24.3/controller.yaml`

You can verify that the controller is running with the following command:
* `kubectl get pods -n kube-system | grep sealed-secrets`
* You should see a pod called `sealed-secrets-controller-...` and the status should be `Running`

### Install the Sealed Secrets CLI
To create a Sealed Secret, you have to install the Sealed Secrets CLI. This can be done with the following command:
* `brew install kubeseal` or from the GitHub [Releases](https://github.com/bitnami-labs/sealed-secrets/releases/tag/v0.24.3)

### Verify the Installation
After the installation, you can verify the installation by running the following command:

```bash
kubectl get pods -n kube-system | grep sealed-secrets
```

If you see the pods of Sealed Secrets, you have successfully installed Sealed Secrets and can proceed with the next steps.

## Step 6: Install a Sample Application
To test the functionality of ExternalDNS, cert-manager and SealedSecrets, we will install a sample application that uses all of these components (our sample application will be the podtato-head application).

### Install the Sample Application
To install the sample application, you can use the following command:

```bash
kubectl apply -f https://github.com/podtato-head/podtato-head-app/releases/download/v0.3.3/manifest.yaml
```

### Delete the old secret
As we will create a new secret, we have to delete the old one. This can be done with the following command:
* `kubectl delete secret podtato-secret`

### Create a Sealed Secret
Now it's time to create a Sealed Secret. This can be done with the following command:
`kubectl create secret generic podtato-secret -n podtato-kubectl --from-literal=PODTATO_SECRET_MESSAGE="Hello Kubernetes Gurus" -oyaml --dry-run | kubeseal --format yaml > podtato-secret.yaml`

After applying the manifest, you should see a Sealed Secret in your cluster:
* `kubectl get sealedsecret podtato-secret`
* and a secret: `kubectl get secret podtato-secret`

### Verify the Installation
After the installation, you can verify the installation by running the following command:

```bash
kubectl get pods -n podtato-kubectl
kubectl get services -n podtato-kubectl
```

If you see the pods of the sample application, you have successfully installed the sample application and can proceed with the next steps.

As you might have noticed, the services are not accessible from the internet yet. Therefore, we will use the NGINX Ingress Controller to make it accessible.

## Step 7: Create an Ingress Resource
To make the sample application accessible from the internet, we have to create an Ingress Resource. This resource will define the rules for the NGINX Ingress Controller to route the traffic to the sample application. Furthermore, we will add the necessary annotations to make the application accessible via HTTPS and to create the necessary DNS records.

### Create the Ingress Resource
To create the Ingress Resource, you can use the following manifest:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: podtato-head-ingress
  namespace: podtato-kubectl
  annotations:
    kubernetes.io/ingress.class: nginx
    cert-manager.io/cluster-issuer: letsencrypt-staging
    external-dns.alpha.kubernetes.io/hostname: podtato-<last-3-digits-of-your-student-id>.uas.on-clouds.at
spec:
  ingressClassName: nginx
  rules:
  - host: podtato-<last-3-digits-of-your-student-id>.uas.on-clouds.at
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: podtato-head-frontend
            port:
              number: 80
  tls:
  - hosts:
      - podtato-<last-3-digits-of-your-student-id>.uas.on-clouds.at
    secretName: podtato-head-tls
```

After you have created the manifest, you can apply it to your cluster with the following command:

```bash
kubectl apply -f <filename>
```

### Verify the Installation
After the installation, you can verify the installation by running the following command:

```bash
kubectl get ingress -n podtato-kubectl
```

After a few moments, you should see the Ingress Resource being created and the NGINX Ingress Controller should create the necessary resources to make the sample application accessible from the internet.

#### Verify the DNS Record
To verify that the DNS record has been created, wait for the instructor to confirm that the DNS record has been created and then run the following command:

```bash
dig podtato-<last-3-digits-of-your-student-id>.uas.on-clouds.at
```

If you see the external IP address of your NGINX Ingress Controller, everything is set up correctly.

### Verify the Certificate
To verify that the certificate has been issued, you can run the following command:

```bash
kubectl get certificates -n podtato-kubectl
```

If you see the certificate being issued, everything is set up correctly.

### Verify the Sample Application
To verify that the sample application is accessible from the internet, you can open a browser and navigate to `https://podtato-<last-3-digits-of-your-student-id>.uas.on-clouds.at`. If you see the podtato-head application, everything is set up correctly.

Although, you have set up a certificate, it is a staging certificate and not trusted by browsers. Therefore, you have to accept the certificate in your browser to access the application.

<aside class="positive">Congrats! You have successfully automated DNS, Certificates and Secrets in Kubernetes!</aside>
