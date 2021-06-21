---
title: "Backup/Restore Stateful App"
draft: false
---

Backup and restore a more complex application, such as MySQL.

The first task is to deploy in OpenShift a MySQL database:

**NOTE:** The storage class used in this example is `ocs-storagecluster-ceph-rbd`
. In case your storage class name is different, change the parameter 
`storageClassName` in the `mysql-persistent-template.yaml` file.

```
$ wget https://gitlab.consulting.redhat.com/iberia-consulting/inditex/ocs/ocs-procedures/-/raw/master/resources/files/oadp/mysql/mysql-persistent-template.yaml -O mysql-persistent-template.yaml
$ oc create -f mysql-persistent-template.yaml
```

This example will create the following OpenShift resources:
* **Namespace:** mysql-persistent
* **Secret:** mysql
* **Service:** mysql
* **PersistentVolumeClaim:** mysql
* **Deployment:** mysql

Before triggering our backup, ensure all these resources have been properly 
created:

```
$ oc get all -n mysql-persistent
NAME                         READY   STATUS    RESTARTS   AGE
pod/mysql-76d9f6cd79-mg7r4   1/1     Running   1          52s

NAME            TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
service/mysql   ClusterIP   172.30.14.89   <none>        3306/TCP   53s

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/mysql   1/1     1            1           52s

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/mysql-76d9f6cd79   1         1         1       52s
```

Once we have verified our deployment is properly set up, we would like to enter 
some data in our MySQL database to ensure the backup and restore process is 
done properly:

```
$ mkdir /tmp/mysql
$ wget http://downloads.mysql.com/docs/world.sql.gz -O /tmp/mysql/world.sql.gz
$ oc rsync /tmp/mysql/ $(oc get pods -o name):/tmp/
$ oc rsh $(oc get pods -o name)
bash$ gzip -d /tmp/world.sql.gz
bash$ mysql -u root
mysql> source /tmp/world.sql;
mysql> use world;
mysql> show tables;
+-----------------+
| Tables_in_world |
+-----------------+
| city            |
| country         |
| countrylanguage |
+-----------------+
3 rows in set (0.00 sec)
```

Once we have data in our MySQL database, trigger the backup in our recently 
created `mysql-persistent` namespace:

```
$ wget https://gitlab.consulting.redhat.com/iberia-consulting/inditex/ocs/ocs-procedures/-/raw/master/resources/files/oadp/mysql/mysql-persistent-backup.yaml -O mysql-persistent-backup.yaml
$ oc create -f mysql-persistent-backup.yaml
```

Make sure the backup is completed:

```
$ oc get backup mysql-persistent -n oadp-operator -o jsonpath='{.status.phase}'
Completed
$ oc get backup mysql-persistent -n oadp-operator -o jsonpath='{.status}' | jq
{
  "completionTimestamp": "2021-02-16T13:04:53Z",
  "expiration": "2021-03-18T13:04:21Z",
  "formatVersion": "1.1.0",
  "phase": "Completed",
  "progress": {
    "itemsBackedUp": 28,
    "totalItems": 28
  },
  "startTimestamp": "2021-02-16T13:04:25Z",
  "version": 1
}
```

Ensure the CSI snapshot has been created properly:

```
$ oc get volumesnapshot -n mysql-persistent 
NAME                 READYTOUSE   SOURCEPVC   RESTORESIZE   SNAPSHOTCLASS                            SNAPSHOTCONTENT                                    AGE
velero-mysql-5pqlf   true         mysql       10Gi          ocs-storagecluster-rbdplugin-snapclass   snapcontent-dcf9a164-f244-45d5-b25f-1a27938fd7bf   75s
```

Once we have ensured the backup is completed, we want to test the restore process. First, delete completely the `mysql-persistent` project:

```
$ oc delete -f mysql-persistent-template.yaml
```

Trigger the restore process of our backup:

```
$ wget https://gitlab.consulting.redhat.com/iberia-consulting/inditex/ocs/ocs-procedures/-/raw/master/resources/files/oadp/mysql/mysql-persistent-restore.yaml -O mysql-persistent-restore.yaml
$ oc create -f mysql-persistent-restore.yaml
```

Verify all our OpenShift resources have been recreated in the restore process:

```
$ oc get all -n mysql-persistent
NAME                         READY   STATUS    RESTARTS   AGE
pod/mysql-76d9f6cd79-mg7r4   1/1     Running   0          46s

NAME            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/mysql   ClusterIP   172.30.166.167   <none>        3306/TCP   44s

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/mysql   1/1     1            1           45s

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/mysql-76d9f6cd79   1         1         1       45s
```

Ensure the restore process has been completed:

```
$ oc get restore -n oadp-operator mysql-persistent -o jsonpath='{.status.phase}'
Completed
$ oc get restore -n oadp-operator mysql-p -o jsonpath='{.status}' | jq
{
  "completionTimestamp": "2021-02-16T13:15:17Z",
  "phase": "Completed",
  "startTimestamp": "2021-02-16T13:14:49Z",
  "warnings": 6
}
```

Ensure the data we have previously imported in our MySQL database is there 
after the restore process:

```
$ oc rsh $(oc get pods -o name)
bash$ mysql -u root
mysql> use world;
mysql> show tables;
+-----------------+
| Tables_in_world |
+-----------------+
| city            |
| country         |
| countrylanguage |
+-----------------+
3 rows in set (0.00 sec)
```
