

oc process -f openshift/deploy.yaml| oc apply -f -


➜  hipster-demo git:(master) ✗ oc process -f openshift/quickstart-netpol.yaml -p NAMESPACE=$(oc project --short=true) | oc apply -f -
networksecuritypolicy.secops.pathfinder.gov.bc.ca/egress-internet created
networksecuritypolicy.secops.pathfinder.gov.bc.ca/intra-namespace-comms created
networksecuritypolicy.secops.pathfinder.gov.bc.ca/int-cluster-k8s-api-comms created


oc delete all -l 'app=hipster-store'


➜  hipster-demo git:(master) ✗ oc get nsp                                                                                            
NAME                        AGE
egress-internet             6m
int-cluster-k8s-api-comms   6m
intra-namespace-comms       6m