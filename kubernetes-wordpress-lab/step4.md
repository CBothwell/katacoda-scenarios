# A New Service for MySQL

This should be mostly a refresher from the first step. Here we are going to create a Service this time for MySQL.

<pre class="file" data-filename="wordpress-config.yml" data-target="append">
---
apiVersion: v1
kind: Service
metadata:
  name: wordpress-mysql
  labels:
    app: wordpress
spec:
  ports:
    - port: 3306
  selector:
    app: wordpress
    tier: mysql
  clusterIP: None
</pre>

We'll add this to our `wordpress-config.yml` and apply the changes.

`kubectl apply -f wordpress-config.yml`{{execute}}

Lets list our services.

`kubectl get service -l app=wordpress`{{execute}}

We should see this new service added to the list of services.