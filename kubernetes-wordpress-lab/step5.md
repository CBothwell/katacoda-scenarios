# A Brief Refelection

By now our `wordpress-config.yml` is getting pretty large. We've got a couple `Services`, a `Secret`, a pair of `PersistantVolumes` and `PersistantVolumeClaims`. Lets take a moment to look at our diagram we saw in the introdution and see what we have left to build. 

![k8s-diagram](k8s-wordpress.png)

As you probably have surmised. We've got everything we need execpt the two deployments. 

# The Deployments

Just like in the other cases, we are going to have a `kind`, Deployment. However, you'll be quick to notice we have a different API this time, `apps/v1`. The status and development of the `Deployment` is governed by a different body than the other objects we've created thus far. You can learn more about this at [API Group Documentation](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/api-machinery/api-group.md)

Our `metadata` should look familiar, however the `spec` is going to be a bit more complex for `Deployments`.

## Containers

`Deployments` wrap `Pods`. The information contained in the `template` works like the definition for the `Pod`. The `template.spec` contains a list of `containers` (Pods have one or more `containers`) and a list of `volumes`. 

For our `continer` we need to specify a Docker image. For the MySQL `Deployment` we are going to use mysql taged with 5.6, `mysql:5.6`. 

In the `env` we are going to create our `MYSQL_ROOT_PASSWORD` using our `Secret`. You an see we use the `Secret's` `name` and `key` defined in the `Secret.data`. 

Additonally we'll create an open port for mysql to listen in on and specifiy a location to mount the `PersistantVolume` at.

## Getting the Volume

We're going to name the `volume` which we already referenced in the `container` and we'll select the `PersistantVolumeClaim` we want to use using its name. 

## The MySQL Config

<pre class="file" data-filename="wordpress-config.yml" data-target="append">
---
apiVersion: apps/v1 
kind: Deployment
metadata:
  name: wordpress-mysql
  labels:
    app: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: mysql
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: wordpress
        tier: mysql
    spec:
      containers:
      - image: mysql:5.6
        name: mysql
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-pass
              key: password
        ports:
        - containerPort: 3306
          name: mysql
        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-persistent-storage
        persistentVolumeClaim:
          claimName: mysql-pv-claim
</pre>

## The WordPress Config

The WordPress config is going to be similar. We need to ensure the correct labels and names are referenced so Kubernetes knows how to link our app together properly.

<pre class="file" data-filename="wordpress-config.yml" data-target="append">
---
apiVersion: apps/v1 
kind: Deployment
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: frontend
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: wordpress
        tier: frontend
    spec:
      containers:
      - image: wordpress:4.8-apache
        name: wordpress
        env:
        - name: WORDPRESS_DB_HOST
          value: wordpress-mysql
        - name: WORDPRESS_DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-pass
              key: password
        ports:
        - containerPort: 80
          name: wordpress
        volumeMounts:
        - name: wordpress-persistent-storage
          mountPath: /var/www/html
      volumes:
      - name: wordpress-persistent-storage
        persistentVolumeClaim:
          claimName: wordpress-pv-claim
</pre>

Okay! Lets add these to our cluster. 

`kubectl apply -f wordpress-config.yml`{{execute}}

Lets make sure things are created properly.

`kubectl get deployment`{{execute}}

`kubectl get all -l tier=mysql`{{execute}}

`kubectl get all -l tier=frontend`{{execute}}

Before moving on we'll want to make sure the status for all deployments is Ready
