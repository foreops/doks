---
title: "Kubernetes Authorization"
description: "A Kubernetes cluster will only authorize your requests after authentication. Authorization grants you access permission for resources in the cluster."
lead: "A Kubernetes cluster will only authorize your requests after authentication. Authorization grants you access permission for resources in the cluster."
date: 2021-06-11T21:31:44-04:00
lastmod: 2021-06-11T21:31:44-04:00
draft: false
weight: 50
images: ["kubernetes-authorization.jpg"]
contributors: []
---

Kubernetes API Server is implemented as a RESTful API service and acts as a front end to its control plane. [REST](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm) is an architecture style developed by [Roy Thomas Fielding](http://www.ics.uci.edu/~fielding/). One of the guiding principles of REST is statelessness. Each request from a client must contain all the information required to complete the request, including authentication and authorization. The RESTful API implementation in Kubernetes makes it compatible with existing on-prem or cloud access systems.  

The API server denies all requests by default. It only authorizes a request when all parts of the request match a policy. You can configure multiple authorization modes. In such a case, it will evaluate all authorization modes one by one until it finds the first authorizer that approves or denies the request, and it immediately returns the result ignoring the rest of the authorizers.

Kubernetes supports several authorization modes:
RBAC
Node
ABAC
Webhook

#### RBAC

Role based access control (RBAC), formalized by [David Ferraiolo and Rick Kuhn](https://csrc.nist.gov/publications/detail/conference-paper/1992/10/13/role-based-access-controls), has been widely adopted by organizations to grant access to systems based on a person's role within the organization or team.  In Kubernetes, roles have a set of permissions tied to them. The permissions match verbs with resources; for instance,  create, get, update, patch, delete with resources pods, services, nodes, etc. The permissions can have a namespace or cluster-wide scope. The API requests made to ```/api/v1/...``` or ```/apis/<group>/<version>/...``` are considered resource requests. Any other requests are "non-resource-requests." They use the HTTP method as the lower case verb. For instance, the GET method becomes the get verb.

RBAC authorization uses the ```rbac.authorization.k8s.io``` API group to drive authorization decisions, allowing you to configure policies through the Kubernetes API dynamically.

There are no "deny" rules in RBAC roles as they are purely additive. An excellent way to think about it is that users, groups, or service accounts are denied access to cluster resources by default, and you explicitly grant access to them. A role does not specify a user or group of users; you essentially assign roles to users or groups, thus creating a binding.  In Kubernetes, you define your RBAC permissions using the following objects:

* ***ClusterRole*** or ***Role*** contains a set of resources and operations that a user or group of users can perform on them. If you want to define a role within a namespace, use the Role object; if you're going to define a role cluster-wide, use a ClusterRole.
* ***ClusterRoleBinding*** or ***RoleBinding*** assigns (binds) a ClusterRole or Role to a user or group of users. A ClusterRoleBinding only is used with a ClusterRole. RoleBinding, on the other hand, will work with either ClusterRole or Role.
ClusterRole allows you to control permissions cluster-wide and have several uses. For instance, grant permissions across all namespaces (for example, all pods cluster-wise), within namespaces, or cluster-wide resources, like nodes. In addition, ClusterRole can be used to control access to non-resources API endpoints, such as ```/healthz```. See [Using RBAC Authorization](https://kubernetes.io/docs/reference/access-authn-authz/rbac/) for various examples.

#### Node

Node authorization mode in Kubernetes authorizes API requests made by kubelets running on nodes. The kubelet is the primary "node agent" that runs on each node and ensures that the containers are running and healthy.  The node authorizer enables kubelets to perform read, write, and authentication-related operations.

#### ABAC

Attribute-based access control (ABAC) is an access control model that consists of rules based on the attributes of the subject (user or group), attributes of the object (pods, namespace), action (read, write), and environmental conditions (nonResourcePath: ```/version```). In addition, ABAC policies express a true/false set that can evaluate many attributes.  Role-based Access Control (RBAC) is a preferred authorization method over the legacy ABAC in Kubernetes. Managing ABAC policies requires one to log into the node to update the file containing ABAC policies. In contrast, administrators can manage RBAC through Kubernetes API.

In the following example, Bob can read pods in the "projectCaribou" namespace.

```yaml
{
  "apiVersion": "abac.authorization.kubernetes.io/v1beta1",
  "kind": "Policy",
  "spec":
    {
      "user": "bob",
      "namespace": "projectCaribou",
      "resource": "pods",
      "readonly": true
    }
}
```

#### Webhook

A webhook is a callback over HTTP. A callback is usually a function passed to another function. The first function calls this function (callback) after it completes.  A more intuitive name for the callback is the "call-after" function. An application implementing webhooks will usually POST  a message to a URL when certain events occur. For examples, when Webhook mode is active, Kubernetes will query an outside RESTful service to determine user privileges. Here is an example of a request the API Server would POST to the remote service for an authorization decision.

```yaml
{
  "apiVersion": "authorization.k8s.io/v1beta1",
  "kind": "SubjectAccessReview",
  "spec": {
    "resourceAttributes": {
      "namespace": "kittensandponies",
      "verb": "get",
      "group": "unicorn.example.org",
      "resource": "pods"
    },
    "user": "jane",
    "group": [
      "group1",
      "group2"
    ]
  }
}
```
The remote service would respond and fill the 'status' field of the request to allow or disallow the request.

```yaml
{
  "apiVersion": "authorization.k8s.io/v1beta1",
  "kind": "SubjectAccessReview",
  "status": {
    "allowed": false,
    "denied": true,
    "reason": "user does not have read access to the namespace"
  }
}
```

#### Key Takeaways

* One of the guiding principles of REST is statelessness. Each request from a client must contain all the information required to complete the request, including authentication and authorization.  
* The API server denies all requests by default. It only authorizes a request when all parts of the request match a policy. 
* Kubernetes will evaluate all configured authorization modes and return immediately as soon as it finds one that approves or denies the request and ignores the rest of the authorizers.
* There are no "deny" rules in RBAC roles as they are purely additive.  The users, groups, or service accounts are denied access to cluster resources by default, and you explicitly grant access to them. 
* A role does not specify a user or group of users; you essentially assign roles to users or groups, thus creating a binding. 
* Node authorization mode in Kubernetes authorizes API requests made by kubelets running on nodes.
* Attribute-based access control (ABAC) is an access control model that consists of rules based on the attributes of the subject (user or group), attributes of the object (pods, namespace), action (read, write), and environmental conditions (nonResourcePath: "/version").
* Managing ABAC policies requires one to log into the node to update the file containing ABAC policies. In contrast, administrators can manage RBAC through Kubernetes API. 
* A webhook is a callback over HTTP. A callback is usually a function passed to another function. The first function calls this function (callback) after it completes; sometimes, it is called a "call-after" function. 
 * When Webhook mode is active, Kubernetes will query an outside RESTful service to determine user privileges. 