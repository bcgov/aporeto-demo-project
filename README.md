

oc process -f openshift/deploy.yaml| oc apply -f -

oc delete all -l 'app=hipster-store'