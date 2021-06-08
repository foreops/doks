---
title: "Kubernetes Authentication"
description: "Authentication in Kubernetes is about verifying the identity of users and services."
lead: ""
date: 2021-06-04T13:12:08-04:00
lastmod: 2021-06-04T13:12:08-04:00
draft: false
weight: 50
images: ["kubernetes-authentication.png"]
contributors: ["Sharjeel Aziz"]
---

Authentication in Kubernetes is about verifying the identity of users and services. This post will discuss authentication methods for two types of accounts that access the Kubernetes cluster: users and machines. While cluster administrators and application developers require access to the cluster to manage and deploy applications respectively, machines, processes, and applications also need access to the cluster, which they obtain through the service accounts.

{{< img src="kubernetes-authentication.png" alt="Kubernetes Authentication" caption="Kubernetes Authentication" class="wide" >}}

### User Authentication

Kubernetes does not have objects which represent user accounts. Users do not log in, and there are no sessions or timeouts. Every request made to the API server is unique, and it contains everything that the API server requires to authenticate/authorize the request. There are many different authentication mechanisms to choose from based on suitability to specific types of implementation.

One of the simplest methods is to use a static token file in CSV format with three required columns: token, user name, user uid, and an optional column for group names. The API server reads bearer tokens from this file which is specified as a command-line option. Updates to the file require access to the node running the API server. Every time the file changes, you have to restart the API server. However, this is not a recommended authentication method as the tokens can last forever.

Users can also authenticate by presenting a certificate signed by the cluster's certificate authority (CA). The user submits the certificate in the form of a Certificate Header or through the kubectl command. The API server reads the username (CN=devuser) and group name (O=engineering) from the 'subject' line of the certificate. When using this method, the administrators are responsible for generating, revoking, and expiry of certificates.

Most of the time, OIDC (OpenID Connect) is used in a production or cloud environment. Users authenticate with their OIDC platform to get tokens. The administrators configure the API server to accept these tokens that contain identity information. It Is also important to note that Kubernetes does not connect to a user directory.

Two more methods are available to use custom identity providers, authenticating proxy or authentication webhook. Authentication proxy used to integrate with LDAP, SAML, Kerberos, alternate x509 schemes, etc. HTTP headers specify a username, group, and any extra information about the user. On the API server, these headers are mapped to the required API server switches.

Webhook authentication allows users to generate tokens through the external service. The users use these tokens when authenticating with the API server. When a client starts to authenticate using a bearer token, the authentication webhook POSTs a JSON-serialized TokenReview object containing the token to the remote service. The remote service indicates success by updating a status field in the request. Usernames derived from various supported authentication identity providers must be unique cluster-wide.

### Machine Authentication

The Service Account controller manages service accounts inside namespaces. It creates a service account named "default" in all active namespaces. When a pod's manifest does not specify a service account,  it will use the "default" service account in its namespace. Service accounts that do not belong to the kube-system namespace have no permissions. Applications access the API server using the service account specified in their pod. An excellent example of such an application is a Kubernetes dashboard that exists in a pod. It will use the service account to talk to the API server. Service accounts use credentials from secrets mounted into pods. Only one service account is per pod is used. Any required roles can be granted to service accounts as needed. Application-specific service accounts should be created and given permissions as needed.

When the API server creates a service account, it generates a token and stores it in a secret object. The API server then links this to the newly created service account. The token in secret is an authentication bearer token used to communicate with the API server. On pods creation, the secret is made available to the pod as a volume.

### Key Takeaways

* Authentication in Kubernetes is about verifying the identity of users (humans) and services (machines, processes, or applications).
Kubernetes does not have objects which represent user accounts. Users do not log in, and there are no sessions or timeouts.
* You do not connect Kubernetes to a user directory. Kubernetes supports several authentication mechanisms out of the box and provides support for custom authentication schemes.
* Usernames derived from various supported authentication identity providers must be unique cluster-wide.
* A service account named "default" exists in all active namespaces.
* Service accounts that do not belong to the kube-system namespace have no permissions.
* Service accounts use credentials from secrets mounted into pods. Each pod can use one service account only.
* When you create pods, the secret in the service account is made available as a volume.
