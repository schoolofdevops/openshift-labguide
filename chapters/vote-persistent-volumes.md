# Steps to set up NFS based Persistent Volumes

Lets first update the default security context for the current namespace to enable containers to run with privileged permissions. This is needed to make sure **postgres** has the permissions to write files to database path, which is restricted by default.

```
oc login -u system:admin

oc adm policy add-scc-to-user privileged -z default

oc login -u developer
oc project instavote

```

In order to use the dynamic provisioning, lets first update the  db deploymentconfig  with *volume* and *volumeMounts* configs as given in example below.



`file: db-dc-pvc.yaml`

```
apiVersion: apps.openshift.io/v1
kind: DeploymentConfig
metadata:
  name: db
  namespace: instavote
spec:
  replicas: 1
  selector:
    tier: back
    app: postgres
  minReadySeconds: 10
  template:
    metadata:
      labels:
        app: postgres
        role: db
        tier: back
        version: "9.4"
    spec:
      containers:
      - image: postgres:9.4
        imagePullPolicy: Always
        name: db
        ports:
        - containerPort: 5432
          protocol: TCP
        securityContext:
          privileged: true
        volumeMounts:
        - name: db-vol
          mountPath: /var/lib/postgresql/data
      #create a volume with pvc
      volumes:
      - name: db-vol
        persistentVolumeClaim:
          claimName: db-pvc
```

Apply *db-dc-pvc.yaml*  as

```
oc apply -f db-dc-pvc.yaml

oc get pod -o wide --selector='role=db'

oc get pvc,pv
```

  * Observe and note which host the pod for *db* is launched.
  * What state is it in ? why?



### Creating a Persistent Volume Claim

switch to project directory

```
cd projects/instavote/dev/
```

Create the following file with the specs below

`file: db-pvc.yaml`

```
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: db-pvc
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 2Gi
  storageClassName: nfs

```


create the Persistent Volume Claim and validate

```
oc get pvc


oc apply -f db-pvc.yaml

oc get pvc,pv

```


## Set up NFS Provisioner in kubernetes

Change into nfs provisioner installation dir

```
cd oc-code/storage
```


Deploy nfs-client provisioner.

```
oc login -u system:admin -n instavote
oc apply -f nfs/

```


This will create all the objects required to setup a nfs provisioner. It would be launched with  Statefulsets. [Read the official documentation on Statefulsets](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/) to understand how its differnt than deployments.


# RBAC Troubleshooting
```
oc get pods
```

[Expected Output]
```
[root@demo-02 storage]# oc get pods
NAME                READY     STATUS    RESTARTS   AGE
nfs-provisioner-0   1/1       Running   0          43s
```

  * Do you see the pod **nfs-provisioner-0** created ? If no, why?  Try to find the root cause.


Solution

```
oc get sts
oc describe sts nfs-provisioner
```
This should tell you that the requested kernel capabilities can not be provided. Why ?

Fix

```
oc adm policy add-scc-to-user privileged -z nfs-provisioner

```

  * How does the above command fix it ? What does it do?



Now lets continue with the persistent volume setup.

```
oc get storageclass
oc get pods
oc logs -f nfs-provisioner-0

```

Now, observe the output of  the following commands,

```
oc get pvc,pv
oc get pods
```

  * Do you see pvc bound to pv ?
  * Do you see the pod for db running ?

Observe the dynamic provisioning, go to the host which is running nfs provisioner and look inside */srv* path to find the provisioned volume.

#### Summary

In this lab, you not only setup dynamic provisioning using NFS, but also learnt about statefulsets as well as rbac policies applied to the nfs provisioner.
