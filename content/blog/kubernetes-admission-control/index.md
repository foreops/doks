---
title: "Admission Control in Kubernetes"
description: "Admission controllers are essential for security, governance, and configuration management in Kubernetes."
lead: "Admission controllers are essential for security, governance, and configuration management in Kubernetes."
date: 2021-06-28T11:00:16-04:00
lastmod: 2021-06-28T11:00:16-04:00
draft: false
weight: 50
images: ["kubernetes-admission-control.jpg"]
contributors: ["Sharjeel Aziz"]
---

 They come in handy to implement or validate resource limits or ensure that the deployments are not using the "latest" image tags. For instance, you can restrict pods to pull container images from certain registries or disallow containers to run in privileged mode. In addition, admission controllers can enforce label naming conventions or add annotations, such as a cost center to objects.

The admission controllers intercept requests made to the API server before the persistence of the object. The API server passes the request to admission controllers after successfully authenticating and authorizing the request. After that, Kubernetes persists everything to [etcd](https://etcd.io/) datastore if admission controllers allow it. etcd is a key/value datastore that stores the current configuration, objects, metadata, and the actual and desired state of the cluster. Kubernetes watches the etcd database for updates and makes any changes if the actual and desired states diverge. That is why admission controllers sit between the persistence of the request and only admitted requests are allowed to change the desired state of the cluster.

```go
// DefaultOffAdmissionPlugins get admission plugins off by default for kube-apiserver.
func DefaultOffAdmissionPlugins() sets.String {
  defaultOnPlugins := sets.NewString(
    lifecycle.PluginName,                    // NamespaceLifecycle
    limitranger.PluginName,                  // LimitRanger
    serviceaccount.PluginName,               // ServiceAccount
    setdefault.PluginName,                   // DefaultStorageClass
    resize.PluginName,                       // PersistentVolumeClaimResize
    defaulttolerationseconds.PluginName,     // DefaultTolerationSeconds
    mutatingwebhook.PluginName,              // MutatingAdmissionWebhook
    validatingwebhook.PluginName,            // ValidatingAdmissionWebhook
    resourcequota.PluginName,                // ResourceQuota
    storageobjectinuseprotection.PluginName, // StorageObjectInUseProtection
    podpriority.PluginName,                  // PodPriority
    nodetaint.PluginName,                    // TaintNodesByCondition
    runtimeclass.PluginName,                 // RuntimeClass
    certapproval.PluginName,                 // CertificateApproval
    certsigning.PluginName,                  // CertificateSigning
    certsubjectrestriction.PluginName,       // CertificateSubjectRestriction
    defaultingressclass.PluginName,          // DefaultIngressClass
  )

  return sets.NewString(AllOrderedPlugins...).Difference(defaultOnPlugins)
}
```

[Code snippet](https://github.com/kubernetes/kubernetes/blob/ea0764452222146c47ec826977f49d7001b0ea8c/pkg/kubeapiserver/options/plugins.go#L141
) from Kubernetes Project

#### Monitoring

As several admission controllers are running,  the API server provides ways to monitor and troubleshoot admission webhooks, for instance, identify webhooks that reject API requests frequently and the reasons for rejection.

#### Dynamic Admission Control

In addition, to the compiled-in admission plugins, custom admission plugins can be implemented as webhooks.  Webhooks are either mutating, validating, or both.  Mutating webhooks modify the objects in an API request, for instance, injecting sidecars into every pod. Validating webhooks validate objects sent to the API server, for example, limiting all replica sets to a minimum of 3.

##### Idempotence

The API server can call an admission controller multiple times.  Idempotence is the property of certain operations in mathematics and computer science whereby they can be applied multiple times without changing the result beyond the initial application. For example, consider that if you are writing an idempotent script to add localhost to /etc/hosts file, you will only append localhost to /etc/hosts file if it does not already exist in the file. Otherwise, if the script added localhost every time it ran, you will have multiple localhost entries.

An idempotent mutating admission webhook can successfully process an object it has already admitted and potentially modified. For example, let say there are two admission webhooks, AlwaysPulIlmage webhook and insert sidecar webhook.  AlwaysPullImage would be called twice, once for the pod object and then after the sidecar injection. As AlwaysPullImage applies to both. On the other hand, you would not want to inject the sidecar every time the API server calls the webhook insert sidecar webhook.

##### Intercepting all versions of an object

As Kubernetes supports multiple [API groups/versions](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.21/#-strong-api-groups-strong-), you can set the .webhooks[].matchPolicy to either Exact or Equivalent when defining a webhook. When matchPolicy is Exact, a webhook would only intercept the request when there is an exact match and miss some if a different version modified the same object. It is best to use Equivalent to match all versions of an object; this ensures that admission controllers continue to work even when resources upgrade to newer versions.

This example shows a validating webhook that intercepts modifications to deployments no matter the API group or version (matchPolicy: Equivalent) and intercepts all apps/v1 Deployment objects:

```yaml
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingWebhookConfiguration
...
webhooks:
- name: my-webhook.example.com
  matchPolicy: Equivalent
  rules:
  - operations: ["CREATE","UPDATE","DELETE"]
    apiGroups: ["apps"]
    apiVersions: ["v1"]
    resources: ["deployments"]
    scope: "Namespaced"
...
...
```

##### Availability

Admission webhooks should evaluate objects, usually in a matter of milliseconds, to ensure that they do not increase latency to API requests. Additionally, you should specify a timeout value so that the request fails after the timeout expires if the webhook is not responding. The default timeout value is 10 seconds and must be between 1 to 30 seconds. Also, the webhook design should ensure high availability and performance; for instance, run several webhook backends behind a service in a cluster.

##### The final state of the object

When implementing a mutating webhook, you can verify the final state of the object by implementing a validating webhook since other webhooks can modify the object after being seen by your mutating webhook.

Avoiding deadlocks when running webhook backends on the same cluster
Deadlocks can occur if a webhook running inside a cluster starts to intercept and reject its resources. For instance, if the node goes down, the scheduler would assign the pods to another node, and the deployment would go through the admission control again. You can run your webhook backend in a separate namespace and then [exclude](https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/#matching-requests-namespaceselector) that namespace.

##### Side-effects

When an API server sends an AdmissionReview "request" to a webhook for review, the webhook responds with an AdmissionReview "response."
The webhook should only make changes to the AdmissionReview object passed to them without any out-of-band ([side-effects](https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/#side-effects)) changes.
If the webhook requires side effects during admission evaluation, it should not make any out-of-band changes if AdmissionReview "request" specifies that it is a dry run.
The API server does not persist dry-run requests; it processes and evaluates the request and returns the final object to the user. The API server only processes dry-run requests if admission controllers explicitly specify that they do not have any side-effects.

```yaml
{
  "apiVersion": "admission.k8s.io/v1",
  "kind": "AdmissionReview",
  "request": {
    # Random uid uniquely identifying this admission call
    "uid": "705ab4f5-6393-11e8-b7cc-42010a800002",

...
...
...
    # dryRun indicates the API request is running in dry run mode and will not be persisted.
    # Webhooks with side effects should avoid actuating those side effects when dryRun is true.
    # See http://k8s.io/docs/reference/using-api/api-concepts/#make-a-dry-run-request for more details.
    "dryRun": false
  }
}
```

AdmissionReview "request"

```yaml
{
  "apiVersion": "admission.k8s.io/v1",
  "kind": "AdmissionReview",
  "response": {
    "uid": "<value from request.uid>",
    "allowed": true
  }
}

```

AdmissionReview "response"

##### Do not operate in the kube-system namespace

Kubernetes manages its components in the kube-system namespace. Therefore, it would be best if you never ran typical workloads in this namespace.  If your webhook mutates or rejects requests in this namespace, it can lead to failure or strange behavior of the control plane.  You can exclude namespaces you do intend to operate in by using the [namespaceSelector](https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/#matching-requests-namespaceselector).

#### Key takeaways

* All requests to the API server pass through admission controllers and if successful persisted to the datastore like etcd.
* Admission has two distinct phases. In the first phase, only mutating admission plugins run. Mutating admission plugins may modify the object that they receive. In the second phase, only validating admission plugins run, and they do not modify the objects.
* A plugin can be a validating plugin, mutating plugin, or both.
* The API server calls mutating admission webhooks one by one while it calls validating admission webhooks in parallel.
* Kubernetes enables a recommended set of around 17 admission plugins by default. Admission controllers are part of the API server binary.
* In addition, to the compiled-in admission plugins, custom admission plugins can be implemented as webhooks.
* The API server can call an admission controller multiple times and should be idempotent.
* It is best to set matchPolicy to Equivalent to match all versions of an object; this ensures that admission controllers continue to work even when resources upgrade to newer versions.
* Admission webhooks should evaluate objects, usually in a matter of milliseconds, to ensure that they do not increase latency to API requests.
* You should run your webhook backend in a separate namespace and then exclude that namespace to avoid deadlocks.
* When an API server sends an AdmissionReview "request" to a webhook for review, the webhook responds with an AdmissionReview "response." The webhook should only make changes to the AdmissionReview object passed to them without any out-of-band (side-effects) changes.
* Do not operate in the kube-system namespace.
