# Adding Openshift Routes


## Setting up Named Based Routing for Vote App

We will direct all our request to the route controller now, but with differnt hostname e.g. **vote.example.com** or **results.example.com**. And it should direct to the correct service based on the host name.

In order to achieve this you, as a user would create a **route** object with a set of rules,


```bash


+----+----+--+            
| router     |            
|            |            
+----+-------+            
     |              +-----+----+
     +---watch----> | route    | <------- user
                    +----------+

```


`file: vote-route.yaml`

```
apiVersion: v1
kind: Route
metadata:
  name: vote
spec:
  host: vote.example.com
  path: "/"
  to:
    kind: Service
    name: vote
```

And apply

```
oc get routes
oc apply -f vote-route.yaml --dry-run
oc apply -f vote-route.yaml
oc get routes
```

Since the router   is constantly monitoring for the route objects, the moment it detects, it connects with traefik and creates a rule as follows.


```bash

                    +------------+
     +--create----> | router     |
     |              |  configs   |
     |              +------------+
+----+----+--+            ^
| route      |            :
| controller |            :
+----+-------+            :
     |              +-----+----+
     +---watch----> | route    | <------- user
                    +----------+

```

where,

  * A user creates a route object with the rules. This could be a named based or a path based routing.
  * An route controller constantly monitors for route objects. The moment it detects one, it creates a rule and adds it to the traefik load balancer. This rule maps to the route specs.



## Adding Local DNS

You have created the route rules based on hostnames e.g.  **vote.example.com**. In order for you to be able to access those, there has to be a dns entry pointing to your nodes, which are running traefik.

```bash

  vote.example.com     -------+                        +----- vote:80
                              |     +-------------+    |
                              |     |   route     |    |
                              +===> |   node:80   | ===+
                              |     +-------------+    |
                              |                        |
  results.example.com  -------+                        +----- results:80

```

To achieve this you need to either,

  * Create a DNS entry, provided you own the domain and have access to the dns management console.
  * Create a local **hosts** file entry. On unix systems its in `/etc/hosts` file. On windows its at `C:\Windows\System32\drivers\etc\hosts`. You need admin access to edit this file.


For example, on a linux or osx, you could edit it as,

```
sudo vim /etc/hosts
```

And add an entry such as ,

```
xxx.xxx.xxx.xxx vote.example.com
```

where,

  * xxx.xxx.xxx.xxx is the actual IP address of one of the nodes running traefik.

And then access the app urls using http://vote.example.com or http://results.example.com

![Name Based Routing](../images/domain-name.png)



## Enabling TLS/SSL Certificates



`file: vote-route.yaml`

```
apiVersion: v1
kind: Route
metadata:
  name: vote
spec:
  host: vote.example.com
  path: "/"
  to:
    kind: Service
    name: vote
  tls:
    insecureEdgeTerminationPolicy: Allow
    termination: edge
```



apply

```
oc apply -f vote-route.yaml
oc get route/vote -o yaml
```
