# Backing up and Restoring etcd Cluster Data

> In a perfect world, infrastructure systems would never have issues. However, when something catastrophic happens, you will likely be happier if you have a backup of your system. In this lesson, we will walk through the process of backing up our etcd cluster data. This will allow us to preserve our cluster state in case the cluster fails and data loss occurs.

### Relevent Docs:
- [Backing Up an etcd Cluster](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster)

---

## Why Back Up etcd?

- `etcd` is the backend data storage solution for your K8s cluster. As such, all your K8s objects, apps, and configs are stored in etcd

- Therefore, you will likely want to be able to back up your cluster's data by backing up etcd, otherwise you will have to rebuild all of your data from scratch by hand

## Backing Up etcd

- You can back up etcd using the etcd command line tool, `etcdctl`

- Use the `etcdctl snapshot save` command to backup the data.
  ```
  $ ETCDTL_API=3 etcdctl --endpoints $ENDPOINT snapshot save <file name>
  ```

## Restoring etcd

- You can restore etcd data from a backup using the `etcdctl snapshot restore` command.

- You will need to supply some additional params, as the restore operation creates a new logical cluster.
  ```
  $ ETCDCTL_API=3 etcdctl snapshot restore <file name>
  ```

# Lab for backup and restore

## Details

> You are working for BeeBox, a subscription service company that provides weekly shipments of bees to customers. The company is using Kubernetes to run some of their applications, and they want to make sure their Kubernetes infrastructure is robust and able to recover from failures.

> Your task is to establish a backup and restore process for the Kubernetes cluster data. Back up the cluster's etcd data, then restore it to verify the process works.

> You can find certificates you can use to authenticate with etcd in /home/cloud_user/etcd-certs.

## Steps

1. set env variable and run the following command:
  ```
  ETCDCTL_API=3 etcdctl get cluster.name \
  --endpoints=https://<private IP address> \
  --cacert=/home/cloud_user/etcd-certs/etcd-ca.pem \
  --cert=/home/cloud_user/etcd-certs/etcd-server.crt \
  --key=/home/cloud_user/etcd-certs/etcd-server.key
  ```
  - Output:
    ```
    cluster.name
    beebox
    ```

2. Perform the backup with:
  ```
  ETCDCTL_API=3 etcdctl snapshot save /home/cloud_user/etcd-backup.db \
  --endpoints=https://<private IP address> \
  --cacert=/home/cloud_user/etcd-certs/etcd-ca.pem \
  --cert=/home/cloud_user/etcd-certs/etcd-server.crt \
  --key=/home/cloud_user/etcd-certs/etcd-server.key
  ```

  - Output:
    ```
    {"level":"info","ts":1621889449.3000798,"caller":"snapshot/v3_snapshot.go:119","msg":"created
    temporary db file","path":"/home/cloud_user/etcd-backup.db.part"}
    {"level":"info","ts":"2021-05-24T20:50:49.306Z","caller":"clientv3/maintenance.go:200","msg":"
    opened snapshot stream; downloading"}
    {"level":"info","ts":1621889449.3061824,"caller":"snapshot/v3_snapshot.go:127","msg":"fetching
    snapshot","endpoint":"https://10.0.1.101:2379"}
    {"level":"info","ts":"2021-05-24T20:50:49.311Z","caller":"clientv3/maintenance.go:208","msg":"
    completed snapshot read; closing"}
    {"level":"info","ts":1621889449.3133507,"caller":"snapshot/v3_snapshot.go:142","msg":"fetched
    snapshot","endpoint":"https://10.0.1.101:2379","size":"20 kB","took":0.013038718}
    {"level":"info","ts":1621889449.3135223,"caller":"snapshot/v3_snapshot.go:152","msg":"saved","
    path":"/home/cloud_user/etcd-backup.db"}
    Snapshot saved at /home/cloud_user/etcd-backup.db
    ```

3. For testing purposes, delete the etcd data:

  ```
  sudo systemctl stop etcd
  ```

  ```
  sudo rm -rf /var/lib/etcd
  ```

4. Restore etcd data from backup previously created:

  ```
  sudo ETCDCTL_API=3 etcdctl snapshot restore /home/cloud_user/etcd-backup.db \
  --initial-cluster etcd-restore=https://10.0.1.101:2380 \
  --initial-advertise-peer-urls https://10.0.1.101:2380 \
  --name etcd-restore \
  --data-dir /var/lib/etcd
  ```

  - Output:

    ```
    {"level":"info","ts":1621890121.5646708,"caller":"snapshot/v3_snapshot.go:296","msg":"restorin
    g snapshot","path":"/home/cloud_user/etcd-backup.db","wal-dir":"/var/lib/etcd/member/wal","dat
    a-dir":"/var/lib/etcd","snap-dir":"/var/lib/etcd/member/snap"}
    {"level":"info","ts":1621890121.5725417,"caller":"membership/cluster.go:392","msg":"added memb
    er","cluster-id":"be98953e0e8f788","local-member-id":"0","added-peer-id":"d961d83f6a817e88","a
    dded-peer-peer-urls":["https://10.0.1.101:2380"]}
    {"level":"info","ts":1621890121.5807483,"caller":"snapshot/v3_snapshot.go:309","msg":"restored
    snapshot","path":"/home/cloud_user/etcd-backup.db","wal-dir":"/var/lib/etcd/member/wal","data
    -dir":"/var/lib/etcd","snap-dir":"/var/lib/etcd/member/snap"}
    ```

5. Un-own etcd from root by running the following:
  ```
  sudo chown -R etcd:etcd /var/lib/etcd
  ```

6. start etcd again:
  ```
  sudo systemctl start etcd
  ```