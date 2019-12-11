# STATE PERSISTENCE

## Volumes

### Sample YAML definition

```
apiVersion: v1
kind: Pod
metadata:
  name: random-number-generator
spec:
  containers:
  - image: alpine
    name: alpine
    command: ["/bin/sh", "-c"]
    args: ["shuf -i 0-100 -n 1 ?? /opt/number.out;"]
    volumeMounts:
    - mountPath: /opt
      name: data-volume
  volumes:
  - name: data-volume
    hostPath:
      path: /data
      type: Directory
```

#### AWS (Amazon Web Services) volume solution

```
  volumes:
  - name: data-volume
    awsElasticBlockStore:
      volumeId: <volume-id>
      fsType: ext4
```

#### [Other volume types](https://kubernetes.io/docs/concepts/storage/#types-of-volumes)

## Persistent Volume

Shorthand for `persistentvolume` is `pv`

### Sample YAML definition

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-vol1
spec:
  accessModes:
  - ReadWriteOnce
  capacity:
    storage: 1Gi
  hostPath:
    path: /tmp/data 
``` 
Note: `hostPath` not to be used in production environment

```
  awsElasticBlockStore:
    volumeId: <volume-id>
    fsType: ext4
```

#### Access Modes
- ReadOnlyMany
- ReadWriteOnce
- ReadWriteMany

#### Creating/Viewing Persistent Volume

```
kubectl create -f pv-definition.yaml
```

```
kubectl get pv
```

## Persistent Volume Claim

Shorthand for `persistentvolumeclaim` is `pvc`

### Sample YAML definition

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: 
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
  persistentVolumeReclaimPolicy: Retain
```

### Creating/Viewing persistent volume claim

```
kubectl create -f pvc-definition.yaml
```

```
> kubectl get pvc 
NAME        STATUS    VOLUME    CAPACITY   ACCESS   STORAGECLASS   AGE
myclaim     Bound     pv-vol1   1Gi        RWO                     43m
```

### [Persistent Volume Reclaim Policy](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#reclaiming):

- Retain (default)

When the PersistentVolumeClaim is deleted, the PersistentVolume still exists and the volume is considered “released”. But it is not yet available for another claim because the previous claimant’s data remains on the volume.

- Delete

For volume plugins that support the Delete reclaim policy, deletion removes both the PersistentVolume object from Kubernetes, as well as the associated storage asset in the external infrastructure, such as an AWS EBS, GCE PD, Azure Disk, or Cinder volume.

- Recycle (deprecated)

If supported by the underlying volume plugin, the Recycle reclaim policy performs a basic scrub (rm -rf /thevolume/*) on the volume and makes it available again for a new claim.
