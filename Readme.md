# Add a Persistent-Volume-Claim to a Kubernetes Dask Cluster
* [Dependencies](#Dependencies)
* [Create the Persistent-Volume](#Create-the-Persistent-Volume)
* [Link persistent-volume with Dask Cluster](#Link-persistent-volume-with-Dask-Cluster)
* [Lunch configuration with helm](#Lunch-configuration-with-helm)

We will see how we can add a Persistent-Volume to a Dask Cluster managed by Kubernetes on local server.

# Dependencies
Before starting we need to make sure we have the following dependencies:

1. minikube: For Lunching a local cluster on our machine.
2. docker :  For Building images necessary to our Dask Cluster.
3. kubectl : A command line tools that lets you control Kubernetes clusters.
4. helm : manages Kubernetes applications
5. Dask Helm charts: install from https://github.com/dask/helm-chart


# Create the Persistent-Volume

We start our local cluster first:
```
minikube start
```

We build then a persistent volume claim (pvc) using a yaml file: `Dask-Persistent-Volume-Claim.yaml`
```
# Dask-Persistent-Volume-Claim.yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: pvc-dask-data # pvc name
spec:
  accessModes:
    - ReadWriteOnce #can be used by a single node -ReadOnlyMany : for multiple nodes -ReadWriteMany: read/written to/by many nodes
  resources: # https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes
    requests:
      storage: 2Gi
```
We lunch then in bash:
```
kubectl apply -f Dask-Persistent-Volume-Claim.yaml
$ persistentvolumeclaim/pvc-dask-data created
```
The pvc creates then a persistent volume, we can check both creation by:
```
kubectl get pvc
```
```
kubectl get pv
```
Now, you should see a pv with Status, capacity,...

# Link persistent-volume with Dask Cluster

We create our Dask Cluser by adding all specs into the yaml file: `values.yaml`

Then we add `volumes` and `volumeMounts` to `jupyter` and `worker` in `values.yaml`

```
  mounts: 
    volumes:
      - name: dask-storage
        persistentVolumeClaim:
         claimName: pvc-dask-data # name of our pvc
    volumeMounts:
      - name: dask-storage
        mountPath: /home/jovyan/save_data # folder for storage
```

# Lunch configuration with helm

We build the dask cluster using helm:
```
helm install my-config dask/dask -f values.yaml
```
Check the creation of the helm configuration:
```
helm list
```
Creating pods can take some time, when finished, we can check the existence of the folder `/home/jovyan/save_data` by connecting to the `jupyter pod`:
```
kubectl exec -ti [pod-jupyter-name] -- /bin/bash
```
where you can get pod name by running
```
kubectl get pods
```
A simple `ls -lt` can show the existence of the folder with relative Read/Write permissions.

When, we finish we can remove our `helm` configuration by:
```
helm delete my-config
```

The pv will persist and be available for further access.