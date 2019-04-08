
# Troubleshooting Security Context Issues


### Enable Images to Run with USER in the Dockerfile
To relax the security in your cluster so that images are not forced to run as a pre-allocated UID, without granting everyone access to the privileged SCC:

Grant all authenticated users access to the anyuid SCC:

$ oc adm policy add-scc-to-group anyuid system:authenticated

###Enable Container Images that Require Root
Some container images (examples: postgres and redis) require root access and have certain expectations about how volumes are owned. For these images, add the service account to the anyuid SCC.



$ oc adm policy add-scc-to-user anyuid system:serviceaccount:myproject:mysvcacct


##### References

  * [Openshift SCC Guide](https://docs.openshift.com/container-platform/3.9/admin_guide/manage_scc.html#enable-images-to-run-with-user-in-the-dockerfile) 
