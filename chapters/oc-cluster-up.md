# Setting up Openshift environment



### Configuring  up Docker Daemon

```
cat <<EOF >> /etc/docker/daemon.json
{
  "insecure-registries" : [
     "172.30.0.0/16"
   ]
}
EOF

```

```
systemctl status docker
systemctl restart docker
systemctl status docker
```


### Install Openshift

Download and setup oc

```
cd /root

wget -c https://github.com/openshift/origin/releases/download/v3.11.0/openshift-origin-server-v3.11.0-0cbc58b-linux-64bit.tar.gz

tar -xzf openshift-origin-server-v3.11.0-0cbc58b-linux-64bit.tar.gz

ln -s /root/openshift-origin-server-v3.11.0-0cbc58b-linux-64bit/oc  /usr/local/bin/

ln -s  ~/openshift-origin-server-v3.11.0-0cbc58b-linux-64bit/oadm /usr/local/bin/
```


Bootstrap an openshift environment

```
oc cluster up --help

oc cluster up --public-hostname=xxxx.xyz.org

```

`replace xxxx.xyz.org with the public hostname of your host`


from web use https://HOST:8443/console (login with user developer, any password)

To login using console

```
oc login -u system:admin
```



### Create and switch to a new Project

```
oc login -u developer

oc projects

oc new-project test

oc projects

oc new-project instavote  --display-name="Instavote" --description="Example Voting App"

oc project instavote

oc config get-contexts


```



## Check out Supporting code

Check out the supporting code from the repository given below. This contains directory structure and supporting YAML files and scaffold useful during this course.

```
git clone https://github.com/schoolofdevops/oc-code.git
cd oc-code
ls
```
