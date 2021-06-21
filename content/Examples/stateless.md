---
title: "Backup/Restore Stateless App"
draft: false
---

Backup and restore a stateless application using OADP/Velero.

The first task is to deploy in OpenShift a nginx application:

```
$ wget https://gitlab.consulting.redhat.com/iberia-consulting/inditex/ocs/ocs-procedures/-/raw/master/resources/files/oadp/nginx/nginx-deployment.yaml -O nginx-deployment.yaml
$ oc create -f nginx-deployment.yaml
```

This example will create the following OpenShift resources:
* **Namespace:** nginx-example
* **Deployment:** nginx-deployment
* **Service:** my-nginx
* **Route:** my-nginx

Before triggering our backup, ensure all these resources have been properly 
created:

```
$ oc get all -n nginx-example
NAME                                    READY   STATUS    RESTARTS   AGE
pod/nginx-deployment-55ddb59f4c-bls2x   1/1     Running   0          2m9s
pod/nginx-deployment-55ddb59f4c-cqjw8   1/1     Running   0          2m9s

NAME               TYPE           CLUSTER-IP      EXTERNAL-IP                                                               PORT(S)          AGE
service/my-nginx   LoadBalancer   172.30.46.198   aef02efae2e95444eaeef61c92dbc441-1447257770.us-east-2.elb.amazonaws.com   8080:30193/TCP   2m10s

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deployment   2/2     2            2           2m10s

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deployment-55ddb59f4c   2         2         2       2m10s

NAME                                HOST/PORT                                                              PATH   SERVICES   PORT   WILDCARD
route.route.openshift.io/my-nginx   my-nginx-nginx-example.apps.cluster-da0d.da0d.sandbox591.opentlc.com          my-nginx   8080   None
```

Once we have verified our deployment is properly set up, trigger the backup in 
our recently created `nginx-example` namespace:

```
$ wget https://gitlab.consulting.redhat.com/iberia-consulting/inditex/ocs/ocs-procedures/-/raw/master/resources/files/oadp/nginx/nginx-stateless-backup.yaml -O nginx-stateless-backup.yaml
$ oc create -f nginx-stateless-backup.yaml
```

Make sure the backup is completed:

```
$ oc get backup -n oadp-operator nginx-stateless -o jsonpath='{.status.phase}'
Completed
$ oc get backup -n oadp-operator nginx-stateless -o jsonpath='{.status}' | jq
{
  "completionTimestamp": "2021-02-15T15:35:59Z",
  "expiration": "2021-03-17T15:35:01Z",
  "formatVersion": "1.1.0",
  "phase": "Completed",
  "progress": {
    "itemsBackedUp": 44,
    "totalItems": 44
  },
  "startTimestamp": "2021-02-15T15:35:01Z",
  "version": 1,
  "warnings": 1
}
```

Once we have ensured the backup is completed, we want to test the restore 
process. First, delete completely the `nginx-example` project:

```
$ oc delete -f nginx-deployment.yaml
```

Trigger the restore process of our backup:

```
$ wget https://gitlab.consulting.redhat.com/iberia-consulting/inditex/ocs/ocs-procedures/-/raw/master/resources/files/oadp/nginx/nginx-stateless-restore.yaml -O nginx-stateless-restore.yaml
$ oc create -f nginx-stateless-restore.yaml
```

Verify all our OpenShift resources have been recreated in the restore process:

```
$ oc get all -n nginx-example 
NAME                                    READY   STATUS    RESTARTS   AGE
pod/nginx-deployment-55ddb59f4c-7dbw7   1/1     Running   0          77s
pod/nginx-deployment-55ddb59f4c-gldml   1/1     Running   0          77s

NAME               TYPE           CLUSTER-IP       EXTERNAL-IP                                                               PORT(S)          AGE
service/my-nginx   LoadBalancer   172.30.158.248   ab58ecf4d417a432792de1219cd3f054-1995587190.us-east-2.elb.amazonaws.com   8080:32036/TCP   76s

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deployment   2/2     2            2           77s

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deployment-55ddb59f4c   2         2         2       77s

NAME                                HOST/PORT                                                              PATH   SERVICES   PORT   WILDCARD
route.route.openshift.io/my-nginx   my-nginx-nginx-example.apps.cluster-da0d.da0d.sandbox591.opentlc.com          my-nginx   8080   None
```

Ensure the restore process has been completed:

```
$ oc get restore -n oadp-operator nginx-stateless -o jsonpath='{.status.phase}'
Completed
$ oc get restore -n oadp-operator nginx-stateless -o jsonpath='{.status}' | jq
{
  "completionTimestamp": "2021-02-15T16:37:19Z",
  "phase": "Completed",
  "startTimestamp": "2021-02-15T16:37:01Z",
  "warnings": 6
}
```
