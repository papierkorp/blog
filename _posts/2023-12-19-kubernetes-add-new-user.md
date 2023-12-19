---
layout: post
title: "How to add Users to a Kubernetes Cluster with a certificate Authentication and export to a kubeconfig file"
date: 2023-12-19
tags: kubernetes howto devops
subtitle: "Step by Step Tutorial on how to add Users to your Kubernetes-Cluster indcluding RBAC and exporting a kubeconfig File"
comments_id: 4
---

# Basic Information for User creation

To create a new User for your Kubernetes Cluster you will need to add 3 things:

1. Some kind of [Authentification](https://kubernetes.io/docs/reference/access-authn-authz/authentication/)
    * X509 certificates
    * static token
    * bootstrap token
    * OpenID Connect
    * Authenticating Proxy
2. [Role / Clusterrole](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#role-and-clusterrole)
    * which [operations](https://kubernetes.io/docs/reference/kubernetes-api/authorization-resources/cluster-role-v1/#Operations) can be done on which [resources](https://kubernetes.io/docs/reference/kubernetes-api/)
    * Role = only counts for one namespace
    * Clusterrole = counts for the whole Cluster
3. [Rolebinding / Clusterrolebinding](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#rolebinding-and-clusterrolebinding)
    * binds the [Subject](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#referring-to-subjects)/Authentication and the Role/Clusterrole together

As it is the easiest and fastest to setup, we will go with the certificate, even tough it might not be the safest option to go. The recommended way would be to use [OpenID Connect](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#openid-connect-tokens).

## create the Authentification

To use certificates as Authentication, we need to first create new Key. Then a CSR (certificate signing request) based on the Private Key and finally authenticate our signing Request to get a Certificate.

```bash
openssl genrsa -out markus.key 2048 #create the private Key
openssl req -new -key markus.key -out markus.csr -subj "/CN=markus/O=testcompany" #create a csr
sudo openssl x509 -req -in markus.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out markus.crt #use the csr to get a Certificate
```

You should savely store the key. While the `.crt` Certificate will be used to authenticate against the cluster.

As far as I know currently we are not able to revoke a once issued Certificate. To deny access to certain User we will go with the deletion of their respective Role/Clusterrole.


## create a Role/Clusterrole

Now we need to create a Role or a Clusterrole for the new User.

To create a User who is allowed to do anything:

```bash
kubectl create clusterrole markus-clusterrole --verb='*' --resource='*'
kubectl edit clusterrole markus-clusterrole #change apiGroups to '*'
```

For the exact possibilies you should take a look at the kubernetes Documentation:

- Verbs: [https://kubernetes.io/docs/reference/kubernetes-api/authorization-resources/cluster-role-v1/#Operations](https://kubernetes.io/docs/reference/kubernetes-api/authorization-resources/cluster-role-v1/#Operations)
- Ressources: [https://kubernetes.io/docs/reference/kubernetes-api/](https://kubernetes.io/docs/reference/kubernetes-api/)
    + Workload: [https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/](https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/)
    + Service: [https://kubernetes.io/docs/reference/kubernetes-api/service-resources/](https://kubernetes.io/docs/reference/kubernetes-api/service-resources/)
    + Configs: [https://kubernetes.io/docs/reference/kubernetes-api/config-and-storage-resources/](https://kubernetes.io/docs/reference/kubernetes-api/config-and-storage-resources/)

If you set up a basic RBAC it can look like this:

```bash
kubectl create clusterrole markus-clusterrole --verb=get,list,watch,create,update,patch,delete --resource=deployments,services,configmaps,secrets
kubectl create role markus-role --verb=create,delete,deletecollection,get,list,patch,update,watch --resource='*'
``` 


If you want to remove access to a certian certificate, you will just have to delete the Role/Clusterrole.

```bash
kubectl delete clusterrole markus-clusterrole
```


## create a Rolebinding/Clusterrolebinding

A Rolebinding/Clusterrolebinding binds the earlier created Role to a Rolebinding/Clusterrolebinding. The Binding also includes a [Subject](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#referring-to-subjects), in our case a `User` (or group, or serviceAccount). 

The user will inherit the role and has to be authenticated outside of Kubernetes. We already created a certificate which we will use as authentication. Everything will be linked together in the [kubeconfig](#combine-the-created-ressources-to-a-kubeconfig-file) File.

```bash
kubectl create clusterrolebinding markus-clusterrolebinding --clusterrole=markus-clusterrole --user=markus
kubectl create rolebinding markus-rolebinding --role=markus-role --user=markus
```


# Basic Information for kubeconfig creation

To create a new kubeconfig File to access your Cluster you need to add 3 parts to your kubeconfig File:

1. cluster
    * a certificate authority as file: `certificate-authority: /etc/kubernetes/pki/ca.crt`
    * or as data: `certificate-authority-data: cat /etc/kubernetes/pki/ca.crt | base64`
2. subject
    * you already learned about the user above
3. context
    * a context binds a user to a cluster and includes a contextName an can include an Namespace


## Combine the created Ressources to a kubeconfig File


```bash
#get the cluster data and save it as a config to a new kubeconfig-markus file
kubectl config set-cluster $(kubectl config view -o jsonpath='{.clusters[0].name}') --server=$(kubectl config view -o jsonpath='{.clusters[0].cluster.server}') --certificate-authority=/etc/kubernetes/pki/ca.crt --kubeconfig=kubeconfig-markus --embed-certs

#get the certificate + key, correctly encode it and save it to the kubeconfig-markus file
#you could also use the path to the key/cert if you leave out --embed-certs=true
kubectl config set-credentials markus --client-certificate=./markus.crt --client-key=./markus.key --embed-certs=true --kubeconfig=kubeconfig-markus

#finally create a context which binds together our user with a cluster and save it to the kubeconfig-markus file
#if you use a role instead of a clusterrole you need to also add --namespace=custom_namespace
kubectl config set-context markus-context --user=markus --cluster=$(kubectl config view -o jsonpath='{.clusters[0].name}') --kubeconfig=kubeconfig-markus

#you can create more than one context, so we need to set the correct context
kubectl config use-context markus-context --kubeconfig=kubeconfig-markus
```


The final result should look like this:

```yaml
apiVersion: v1
clusters:
  - cluster:
      certificate-authority-data: LS0tLS1CRU....LS0K
      server: https://cluster-name-api.company.de:6443
    name: cluster-name
contexts:
  - context:
      cluster: cluster-name
      user: markus
    name: cluster-name-markus
current-context: cluster-name-markus
kind: Config
preferences: {}
users:
  - name: markus
    user:
      client-certificate-data: LS0tLS1CRU...UtLS0tLQo=
      client-key-data: LS0tLS1C....S0tLS0K
```


Now a simple test if everything works:

```bash
k get nodes --kubeconfig="kubeconfig-markus"
```

## configure and use kubeconfig locally

If you dont use Linux its hardly recommended to use a WSL (Windows Subsystem Linux) Shell. For this go to `Windows-Features` and activate `Windows-Subsystem for Linux`, afterwards install `Ubuntu` via the Microsoft Store.

Now we need to edit the `.bashrc`:

```bash
vi ~/.bashrc
export KUBECONFIG="${KUBECONFIG}:/mnt/c/develop/kubeconfig_cluster1.yml:/mnt/c/develop/kubeconfig_cluster2.yml"
source ~/.bashrc
```

And if you want, here some useful Aliase:

```bash
vi ~/.bash_aliases
alias k=kubectl
alias usecluster1="kubectl config use cluster-name-markus"
alias usecluster2="kubectl config use cluster-name-markus2"
alias ccon="kubectl config current-context"
source ~/.bashrc
```

Now how do you use different contexts for different cluster:

```bash
#show the current context = the current cluster
kubectl config current-context
#show all available contexts
kubectl config get-contexts
#use a specific context with a specific user
kubectl config use-context user@cluster-name
```


# Afterwords

I hope this is useful for some of you, if you have question dont hesitate to ask in the Comment section.

Im also still in the learning phase of Kubernetes, so Im open for suggestions for improvement.