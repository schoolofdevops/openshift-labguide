



## creating security context

oc login -u system:admin -n default
oc get scc

#oc apply -f instavote-scc.yaml
#oadm policy add-scc-to-user instavote developer

#oc adm policy add-scc-to-user anyuid developer



oc login -u system:admin -n instavote
oc adm policy add-scc-to-user privileged -z default

oc adm policy add-scc-to-user privileged -z system:serviceaccount:instavote:default



$ oc adm policy add-scc-to-user anyuid system:serviceaccount:myproject:mysvcacct



### Users Management


### Administration

Idling an application stops the pod and will launch again when there is traffic received on the corresponding service.

```
oc idle vote
```
