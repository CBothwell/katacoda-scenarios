# Add Some PersistantVolumes

For MySQL we need some disk storage to keep our data on. Similarly, if we want to add plugins or themes to WordPress we are going to need some place that those files can be written to. 

To facilite that, we're going to look at our next object or `kind`, the `PersistantVolume`.

For the `spec` there are a couple of parameters we need to think about when we create the object. 

The `capacity` will let us specify how much space we get.

The `storageClassName` will let us set the storage class we are going to use. Storage classes are vender specific. For AWS we might have `AWSElasticBlockStore`, for Azure we might have `AzureFile`. In our case, we are going to use the `manual` and specify a location on the node using `hostPath`. This is **not** a production storage class. Each of these may support one or more `accessModes`. 

`accessModes` let us list the types of access a claim can make a `PersistantVolume`

+ ReadWriteOnce
+ ReadOnlyMany
+ ReadWriteMany

In our case, `hostPath` only supports `ReadWriteOnce`, which is fine for our purposes.

Additionally, we need to specify the `persistentVolumeReclaimPolicy`. It can be one of the following. 

+ Retain 
+ Recycle (will delete the contents of the volume) -- Note this is depricated
+ Delete (will delete the storage asset)

Lastely, we'll need to do a bit of preplanning here. To get these to link up with our specific VoluemeClaims, we'll need to add a new label which we can use to select the specific Volume.

Armed with that information lets go ahead and create our two PersistantVolumes:

<pre class="file" data-filename="wordpress-config.yml" data-target="append">
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv
  labels:
    app: wordpress
    tier: mysql
spec:
  capacity:
    storage: 5Gi 
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  hostPath: 
    path: "/mnt/data"
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: wordpress-pv
  labels:
    app: wordpress
    tier: frontend
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  hostPath: 
    path: "/mnt/frontend"
</pre>

Now lets apply those changes to our cluster. 

`kubectl apply -f wordpress-config.yml`{{execute}}

We should see that the output creates the two volumes, but that the existing service is unchanged.

Now lets look at our created volumes. First the WordPress volume.

`kubectl get pv/wordpress-pv`{{execute}}

As you can see, we can get the individual resource by prefixing the type (pv for persistant volume) and then `/` with the resource's name. 

Now let look at the MySQL volume.

`kubectl get pv/mysql-pv`{{execute}}

All volumes at the same time.

`kubectl get pv`{{execute}}

We should see both volumes listed. Now lets see all of the resources for our app.

`kubectl get all -l app=wordpress`{{execute}}

# Add Some PersistantVolumeClaims

Just like the `PersistantVolumes` we need to do a bit of leg work to make sure these get set up correctly.

The `kind` for this is, unsurprisingly, `PersistentVolumeClaim`. We will include our bit of standard `metadata` and take a look at the `spec`.

We need to select an `accessMode` that corrisponds to one of the avaiable for the volume. We'll want to also include the same `storageClassName` and the same size of storgae (or less) under the `resources.requests.storage`.

The last important bit for these two volume claim is the `matchLabel` selector. As you may recall we added another label in the `metadata` of our `PersistantVolumes` we can use this label to select which claim goes where. Otherwise, Kubernetes will auto assign these out based on the other values in the `spec` with what matches the availiable `PersistantVolumes`. 

The config changes should look like:

<pre class="file" data-filename="wordpress-config.yml" data-target="append">
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: wordpress-pv-claim
  labels:
    app: wordpress
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: manual
  resources:
    requests:
      storage: 5Gi
  selector:
    matchLabels:
      tier: frontend
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
  labels:
    app: wordpress
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: manual
  resources:
    requests:
      storage: 5Gi
  selector:
    matchLabels:
      tier: mysql
</pre>

Lets apply those changes to our cluster. 

`kubectl apply -f wordpress-config.yml`{{execute}}

Now lets take a look at the volume claims.

`kubectl get pvc`{{execute}}

We should see both claims created and bound. And we should should similar detail for the persistant volumes.

`kubectl get pv`{{execute}}

We should see both volumes claimed with the correctly named volume claim.