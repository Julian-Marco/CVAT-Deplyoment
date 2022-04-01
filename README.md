# CVAT-Deplyoment

This document is supposed to present our current approach for deploying CVAT (more or less) succesfully to the OKD cluster.

## Requirements

1. Openshift command line interface (CLI) `oc`.
2. Openshift's administrative rights to create new projects and make deployments on the created project (namespace).

## Setup

1. Open your command line interface and apply the following command:

```
helm install -n  <namespace> <release_name> --create-namespace ./helm-chart -f ./helm-chart/values.yaml
```

2. If the deployment went successfully, switch to the project folder by typing the following:

```
oc project <namespace>
```

3. We need to take care of some security issues from here on. The first step is to change the security context constraints of the whole project:

```
oc adm policy add-scc-to-user privileged -z default -n <release_name>
```
Now switch to the project folder that lies within the OKD cluster and click on "Deployments". You'll see three different deployments called "cvat-backend", "cvat-frontend" and "cvat-opa". At this point, we have yet to find out how to repair "cvat-opa", but in order to repair both "cvat-backend" and "cvat-frontend", we can do the  following:

4. Click on the respective YAML file. Again, we want to modify the security context constraints by adding this line
```
openshift.io/scc: privileged
```
within the context of 
```
metadata:
  annotations:
    deployment.kubernetes.io/revision: '3'
    meta.helm.sh/release-name: cvat
    meta.helm.sh/release-namespace: cvat-docu-test
    openshift.io/scc: privileged
```
Furthermore, we need to add
```
securityContext:
        runAsUser: 0
        runAsGroup: 0
```
within the context of
```
restartPolicy: Always
terminationGracePeriodSeconds: 30
dnsPolicy: ClusterFirst
securityContext:
  runAsUser: 0
  runAsGroup: 0
schedulerName: default-scheduler
```
Ideally, we also change
```
imagePullPolicy: Always
```
to
```
imagePullPolicy: IfNotPresent
```
Restarting the respective Deployments should now yield a running cvat-backend and cvat-frontend pod.
