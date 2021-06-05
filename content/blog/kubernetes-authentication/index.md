---
title: "Kubernetes Authentication"
description: ""
lead: ""
date: 2021-06-04T13:12:08-04:00
lastmod: 2021-06-04T13:12:08-04:00
draft: false
weight: 50
images: ["kubernetes-authentication.png"]
contributors: ["Sharjeel Aziz"]
---

In this post, we will discuss two types of accounts that access the Kubernetes cluster. Users and machines.  Authentication in Kubernetes is about verifying the identity of users and services. Administrators manage the cluster. Developers deploy applications to the cluster. End-users use applications deployed on the cluster. They authenticate with the application running on the cluster. They do not need access to the cluster. Cluster administrators and application developers do need access to the clusters. Machines, processes, or applications also need access to the cluster. They use service accounts to access the cluster.

{{< img src="kubernetes-authentication.png" alt="Kubernetes Authentication" caption="Kubernetes Authentication" class="wide" >}}

### How do users connect to the cluster?

Kubernetes does not have objects which represent user accounts. Users do not log in, and there are no sessions or timeouts. Every request made to the API server is unique. It contains everything that the API server requires to authenticate/authorize the request.  You can choose many authentication mechanisms that are suitable for your implementation.

One of the simplest methods is to use a Static Token file. The API server reads bearer tokens from this file that you specify as a command-line option. It is not a recommended method as the tokens can last forever. You need to access the node running the API server to update the file. Every time you make changes to the file, you have to restart the API server. The token file is a csv file with three columns: token, user name, user uid, and optional group names.

Users can authenticate by presenting a certificate signed by the cluster's certificate authority (CA). The user submits the certificate in the form of a Certificate Header or through the kubectl command. The API server reads the username (CN=devuser) and group name (O=engineering) from the ‘subject’ line of the certificate. When using this method, the administrators are responsible for generating, revoking, and expiry of certificates.

Most of the time, you would be using OIDC (OpenID Connect) in a production or cloud environment. Users authenticate with their OIDC platform to get tokens. The administrators configure the API server to accept these tokens that contain identity information. An important concept to understand is that you do not connect Kubernetes to a user directory.

Two more methods are available if you need to use custom identity providers. Using an authenticating proxy or authentication webhook, you can integrate with LDAP, SAML, Kerberos, alternate x509 schemes, etc.). You use HTTP headers to specify a username, group, and any extra information about the user. On the API server, you map these headers to the required API server switches.

Webhook authentication allows your users to generate tokens through the external service. The users use these tokens when authenticating with the API server. When a client starts to authenticate using a bearer token, the authentication webhook POSTs a JSON-serialized TokenReview object containing the token to the remote service. The remote service indicates success by updating a status field in the request.  Usernames derived from various supported authentication identity providers must be unique cluster-wide.


### How do things running inside the cluster interact with each other?

The Service Account controller manages service accounts inside namespaces. It creates a service account named “default” in all active namespaces. When you create a pod without a service account, it will use the “default” service account in the namespace. Service accounts that do not belong to the kube-system namespace have no permissions. Application access the API server using the service account specified in their pod. An excellent example of such an application is a Kubernetes dashboard that exists in a pod. It will use the service account to talk to the API server. Service accounts use credentials from secrets mounted into pods. You can only use one service account per pod. You can specify a service account in the pods manifest. You can grant particular roles to service accounts as needed. You should create application-specific service accounts and then give them permissions as required.

When the API server creates a service account, it generates a token and stores it in a Secret object. The API server then links this to the newly created service account. The token in secret is an authentication bearer token used to communicate with the API server. When you create pods, this secret is made available to the pod as a volume.


### Key Takeaways

*   Authentication in Kubernetes is about verifying the identity of users (humans) and services (machines, processes, or applications).
*   Kubernetes does not have objects which represent user accounts. Users do not log in, and there are no sessions or timeouts.
*   You do not connect Kubernetes to a user directory.
*   Kubernetes supports several authentication mechanisms out of the box and provides support for custom authentication schemes.
*   Usernames derived from various supported authentication identity providers must be unique cluster-wide.
*   A service account named “default” exists in all active namespaces.
*   Service accounts that do not belong to the kube-system namespace have no permissions.
*   Service accounts use credentials from secrets mounted into pods. Each pod can use one service account only. 
*   When you create pods, the secret in the service account is made available as a volume.