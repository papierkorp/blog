---
layout: post
title: "How to add Users to a Kubernetes Cluster with a certificate Authentication"
date: 2023-09-18
tags: kubernetes howto devops
subtitle: "Step by Step Tutorial on how to add Users to your Kubernetes-Cluster indcluding RBAC and exporting a kubeconfig File"
---

http://localhost:8800/doku.php?id=sirconic:systemadmin:kubernetes#kubeconfig
https://gauravwadghule.medium.com/how-to-grant-users-to-access-the-kubernetes-cluster-with-a-client-certificate-a1763db2e74a



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

- Verbs: https://kubernetes.io/docs/reference/kubernetes-api/authorization-resources/cluster-role-v1/#Operations
- Ressources: https://kubernetes.io/docs/reference/kubernetes-api/
    + Workload: https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/
    + Service: https://kubernetes.io/docs/reference/kubernetes-api/service-resources/
    + Configs: https://kubernetes.io/docs/reference/kubernetes-api/config-and-storage-resources/

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

This user has to be authenticated outside of Kubernetes. We already created a certificate which we will use as authentication. Everything will be linked together in the [kubeconfig](#Combine-the-created-Files-to-a-kubeconfig-File) File.



# Basic Information for kubeconfig creation

To create a new kubeconfig File to access your Cluster you need to add 3 parts to your kubeconfig File:

1. cluster
    * a certificate authority as file: `certificate-authority: /etc/kubernetes/pki/ca.crt`
    * or as data: `certificate-authority-data: cat /etc/kubernetes/pki/ca.crt | base64`
2. subject
    * you already learned about the user above
3. context
    * a context binds a user to a cluster and includes a contextName an can include an Namespace

d
d
d
d

d
d
d
d
d
d

d
d
d
d

d
d
d
d
d
d

d

d
d
d
d

d
d
d
d
d
d

d
d
d
d
d

d
d
d
d
d
d

d

d
d
d
d

d
d
d
d
d
d

d
d
d
d
d

d
d
d
d
d
d

d
d
## Combine the created Files to a kubeconfig File