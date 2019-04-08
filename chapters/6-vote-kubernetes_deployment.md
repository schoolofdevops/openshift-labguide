# LAB K106 - Defining Release Strategy with  deploymentconfig

A deploymentconfig is a higher level abstraction which sits on top of replica sets and allows you to manage the way applications are deployed, rolled back at a controlled rate.

deploymentconfig has mainly three responsibilities,

  * Availability: Maintain the number of replicas for a type of service/app.
  * Scalability: Schedule/delete pods to meet the desired count.
  * Update Strategy: Define a release strategy and update the pods accordingly.

```
oc projects
oc project instavote

cd projects/instavote/dev/
cp vote-rc.yaml vote-dc.yaml
```


[deploymentconfig spec](https://docs.okd.io/latest/dev_guide/deployments/deployment_strategies.html)  contains everything that replica set has + strategy. Lets add it as follows,

`file: vote-dc.yaml`

```
apiVersion: apps.openshift.io/v1
kind: DeploymentConfig
metadata:
  name: vote
spec:
  minReadySeconds: 20
  replicas: 12
  selector:
    role: vote
  strategy:
    type: Rolling
    rollingParams:
      updatePeriodSeconds: 1
      intervalSeconds: 1
      timeoutSeconds: 120
      maxSurge: 2
      maxUnavailable: 1
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
          image: initcron/oc-vote:v1
          resources:
            requests:
              memory: "64Mi"
              cpu: "50m"
            limits:
              memory: "128Mi"
              cpu: "250m"
```


This time, start monitoring with --show-labels options added.

```
watch -n 1 oc get  dc,rc,pods --show-labels
```


Lets  create the deploymentconfig. Do monitor the labels of the pod while applying this.

```
oc apply -f vote-dc.yaml
```


Now that the deploymentconfig is created. To validate,

```
oc get dc
oc get rc --show-labels
oc get deploy,pods,rc
oc rollout status dc  vote
oc get pods --show-labels
```

Also apply the service spec created earlier.
```
oc apply -f vote-svc.yaml
oc get endpoints
```
Sample Output
```
oc get dc
NAME       DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
vote   3         3         3            1           3m
```


## Rolling Updates in Action

Now, update the deploymentconfig spec to apply

file: vote-dc.yaml
```

...
template:
  metadata:
    labels:
      version: v2   
  spec:
    containers:
      - name: app
        image: initcron/oc-vote:v2

```

apply

```
oc apply -f vote-dc.yaml

oc rollout status deploymentconfig/vote
```

Observe rollout status and monitoring screen.



```

oc rollout history dc vote

oc rollout history dc vote --revision=1

oc rollout history dc vote --revision=1
```

Try updating the version of the image from v2 to v3,v4,v5. Repeat a few times to observe how it rolls out a new version.  

## Undo and Rollback

Lets try to introduce an error by providing a non existing image. To make an ad hoc change, we would use edit commnd this time.

```
oc edit dc vote
```

and update the image to something similar to following,

```
spec:
  containers:
    - name: app
      image: initcron/oc-vote:rgjerdf

```

apply

```
oc apply -f vote-dc.yaml

oc rollout status dc vote

oc rollout history dc vote

oc rollout history dc vote --revision=xx
```

where replace xxx with revisions

Find out the previous revision with sane configs.

To undo to a sane version (for example revision 3)

```
oc rollout undo dc vote --to-revision=2
```


##### References

  * [deploymentconfig documentation](https://docs.okd.io/latest/dev_guide/deployments/deployment_strategies.html)
  * [deploymentconfig api refence](https://docs.okd.io/latest/rest_api/apis-apps.openshift.io/v1.DeploymentConfig.html)
