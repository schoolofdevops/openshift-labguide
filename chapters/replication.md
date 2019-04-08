
# Lab K104 - Adding HA and Scalability with Replication Controllers

If you are not running a monitoring screen, start it in a new terminal with the following command.

```
watch -n 1 kubectl get  pod,rc
```

## Adding Replication Controller  Configurations

Lets now write the spec for the Replication Controller . This is going to mainly contain,

  * replicas
  * selector
  * template (pod spec )
  * minReadySeconds


From here on, we would switch to the project and environment specific path and work from there.


```
cd projects/instavote/dev

```


`edit file: vote-rc.yaml`

```
apiVersion: xxx
kind: xxx
metadata:
  xxx
spec:
  xxx
  template:
    metadata:
      name: vote
      labels:
        app: python
        role: vote
        version: v1
    spec:
      containers:
        - name: app
          image: schoolofdevops/vote:v1
          resources:
            requests:
              memory: "64Mi"
              cpu: "50m"
            limits:
              memory: "128Mi"
              cpu: "250m"
```

Above file already containts the spec that you had written for the pod. You would observe its already been added as part of *spec.template* for Replication Controller.

Lets now add the details specific to Replication Controller.

*file: vote-rc.yaml*

```
apiVersion: apps/v1
kind: ReplicationController
metadata:
  name: vote
spec:
  minReadySeconds: 20
  replicas: 4
  selector:
    role: vote
  template:
    metadata:
      name: vote
      labels:
        app: python
        role: vote
        version: v1
    spec:
      containers:
        - name: app
          image: schoolofdevops/vote:v1
          resources:
            requests:
              memory: "64Mi"
              cpu: "50m"
            limits:
              memory: "128Mi"
              cpu: "250m"
```

The complete file will look similar to above. Lets now go ahead and apply it.


```
kubectl apply -f vote-rc.yaml --dry-run

kubectl apply -f vote-rc.yaml

kubectl get rc

kubectl describe rc vote

kubectl get pods

kubectl get pods --show-labels
```

### High Availability

Try deleting pods created by the Replication Controller,

`replace pod-xxxx and pod-yyyy with actuals`
```
kubectl get pods

kubectl delete pods vote-xxxx vote-yyyy
```
Observe as the pods are automatically created again.


Lets now delete the pod created independent of replication  controller.

```
kubectl get pods
kubectl delete pods  vote
```

Observe what happens.
  * Does replica set take any action after deleting the pod created outside of its spec ? Why?

### Exercise: Deploying new version of the application


```
kubectl edit rc/vote
```

Update the version of the image from **schoolofdevops/vote:v1** to **schoolofdevops/vote:v2**

Save the file.

Observe what happens ?

  * Did application get  updated.
  * Did updating Replication Controller launched new pods to deploy new version ?


### Scalability

Scaling up application is as easy as running,  

```
kubectl scale --replicas=8 rc/vote

kubectl get pods --show-labels
```  

Observe what happens

  * Did the number of  replicas increase to 8 ?
  * Which version of the app are the new pods running with ?


#### Summary

With **Replication Controllers** your application is now high available as well as scalable. However Replication Controller by itself does not have the intelligence to trigger a rollout if you update the version. For that, you are going to need a **deployment** which is something you would learn in an upcoming  lesson.
