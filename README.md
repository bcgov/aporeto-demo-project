# TL;DR

Check out the manifests in `openshift/`. There you'll find one to deploy the Hipster Store demo application and the associated Network Security Policy (NSP) to make it work.

# Introduction

This project was created to show off some best practices for labeling your application, components, routes, and network security policy. Labeling is is key to both a healthy easy to manage application as well as implementing meaningful custom NSP.

The Hipster Store (HS) is a sample project created by Google to illustrate a microservices architecture designed to run on [Kubernetes](Kubernetes.io) (k8s). The HS makes for a great teaching application for Aporeto for a few reasons: its has lots of components with specific communication paths; its a working application; and its well documented.

As part of this demo project the HS application was converted to work on OCP and custom NSP was created to only allow components talk to the other components required to preform their job according to the [Service Architecture](https://github.com/GoogleCloudPlatform/microservices-demo#service-architecture). 

# Labels

Labels serve many purposes in the OpenShift Container Platform (OCP). Applying labels to pods or routes behavior can be assign or changed. In our case, labels are used to create an identity for the HS application, as a whole, as well as its constituent component.

# Application Identity

Build the identity of our application and components by applying labels at different levels:

The first label is attributed to the `Deployment` section of each component in our [app.yaml](openshift/app.yaml) manifest. By inserting the label `app: hipster-store` at this level we group all the components into a single application visible in the OCP Web console.

```yaml
- apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: emailservice
    labels:
      app: hipster-store
  spec:
```

This results in all of the components grouped within OCP as seen by the following web console screen-shot:

![Application Labels](./images/application-group.png)

The second label is attributed to the `template` section of the manifest. This unique label is applied to the running Pod and it the secret sauce in our identity. 

```yaml
template:
  metadata:
  labels:
    app: hipster-store
    role: emailservice
```

This results in unique identifying a component by its role within OCP as seen by the following web console screen-shot.

![Email Service Pod](./images/email-service-pod.png)

**🤓 ProTip**

Pods are also called Processing Units (PU) in Aporeto parlance; notice how the PU
inheres the `app: hipster-store` label from its deployment config. You can use this in NSP to reference all components. This would work well for your Pods-to-k8s-API policy.

# Run & Go

## Deploy It

## Cleanup



oc process -f openshift/deploy.yaml| oc apply -f -


➜  hipster-demo git:(master) ✗ oc process -f openshift/quickstart-netpol.yaml -p NAMESPACE=$(oc project --short=true) | oc apply -f -
networksecuritypolicy.secops.pathfinder.gov.bc.ca/egress-internet created
networksecuritypolicy.secops.pathfinder.gov.bc.ca/intra-namespace-comms created
networksecuritypolicy.secops.pathfinder.gov.bc.ca/int-cluster-k8s-api-comms created


➜  hipster-demo git:(master) ✗ oc process -f openshift/loadgen.yaml -p NAMESPACE=$(oc project --short=true) | oc apply -f - 
networksecuritypolicy.secops.pathfinder.gov.bc.ca/loadgen-to-frontent created
deployment.apps/loadgenerator created



➜  aporeto-demo-project git:(master) ✗ oc delete all,nsp -l "app=hipster-store"
pod "adservice-655ccdfc9-b6jvd" deleted
pod "cartservice-698d86d5b4-cb4bc" deleted
pod "checkoutservice-7c6d6f5c79-vtwms" deleted
pod "currencyservice-6bc7895789-s57f7" deleted
pod "emailservice-64cb7dccdc-js2fj" deleted
pod "frontend-794f4f65c9-sww84" deleted
pod "loadgenerator-6b4664dd7-8kf48" deleted
pod "paymentservice-58565cc5d7-769nk" deleted
pod "productcatalogservice-66596db955-dq94d" deleted
pod "recommendationservice-5585758bb7-z6z6c" deleted
pod "redis-cart-fc9dfd764-lk8sz" deleted
pod "shippingservice-75756bfd9f-2m2wk" deleted
service "adservice" deleted
service "cartservice" deleted
service "checkoutservice" deleted
service "currencyservice" deleted
service "emailservice" deleted
service "frontend" deleted
service "paymentservice" deleted
service "productcatalogservice" deleted
service "recommendationservice" deleted
service "redis-cart" deleted
service "shippingservice" deleted
deployment.apps "adservice" deleted
deployment.apps "cartservice" deleted
deployment.apps "checkoutservice" deleted
deployment.apps "currencyservice" deleted
deployment.apps "emailservice" deleted
deployment.apps "frontend" deleted
deployment.apps "loadgenerator" deleted
deployment.apps "paymentservice" deleted
deployment.apps "productcatalogservice" deleted
deployment.apps "recommendationservice" deleted
deployment.apps "redis-cart" deleted
deployment.apps "shippingservice" deleted
route.route.openshift.io "frontend-route" deleted
networksecuritypolicy.secops.pathfinder.gov.bc.ca "cartservice-to-int-services" deleted
networksecuritypolicy.secops.pathfinder.gov.bc.ca "checkoutservice-to-int-services" deleted
networksecuritypolicy.secops.pathfinder.gov.bc.ca "frontend-to-int-services" deleted
networksecuritypolicy.secops.pathfinder.gov.bc.ca "int-cluster-k8s-api-comms" deleted
networksecuritypolicy.secops.pathfinder.gov.bc.ca "loadgen-to-frontend" deleted
networksecuritypolicy.secops.pathfinder.gov.bc.ca "recommendationservice-to-int-services" deleted
➜  aporeto-demo-project git:(master) ✗ 


➜  aporeto-demo-project git:(master) ✗ oc scale deployment loadgenerator --replicas=0 
deployment.extensions/loadgenerator scaled

➜  aporeto-demo-project git:(master) ✗ oc delete nsp checkoutservice-to-int-services
networksecuritypolicy.secops.pathfinder.gov.bc.ca "checkoutservice-to-int-services" deleted

➜  aporeto-demo-project git:(master) ✗ oc get pods,dc,services,routes
No resources found.

➜  hipster-demo git:(master) ✗ oc get nsp                                                                                            
NAME                        AGE
egress-internet             6m
int-cluster-k8s-api-comms   6m
intra-namespace-comms       6m



https://github.com/GoogleCloudPlatform/microservices-demo

Hipster Store Service Architecture
https://github.com/GoogleCloudPlatform/microservices-demo#service-architecture


NOTES:
- Adding `- - $namespace=${NAMESPACE}` to both source and destination is important because, in theory, on OCP4 someone
could create a pod with matching labels that can talk to other
peoples pods.
- You get pod crashes if, when they start, they can't talk to one another.
- The app does not seem to recover after policy is added. Need to re-deploy the application.

### Add policy to:

- [x] Pods to internal k8s API. (Done)
- [x] Internet to Frontend. (Not Required ATM due to inherited rules)
- [x] Frontend to (AdService, CheckoutService, ShippingService, CurrencyService, ProductCatalogService, RecommendationService, CartService)
- [x] Checkout Service to EmailService, PaymentService, ShippingService, CurrencyService, ProductCatalogService, CartService.
- [x] RecommendationService to ProductCatalogService
- [x] CartService to Cache (redis)


STEPS:

1. Convert labeling scheme to be NSP focused by using `app` to group the 
hipster-store components to an application and `role` to specify what the
component (pod) does. This will allow us to run other applications in the same
namespace (if needed) and use `role=` to identify processing units (PU) a.k.a pods in our NSP.