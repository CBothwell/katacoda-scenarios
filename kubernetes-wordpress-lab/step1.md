# Set up a Service

The Service is going to be our first Kubernetes object that we create. To create any object we need to know the API that it is defined in. In our cases the Service is defined in the core API, `v1`. 

We also need to know the `kind`, you can think of this as being similar to the class. Kubernetes provieds a number of *primative* `kind`s defined in different API groups.

After the API and Kind are defined we need to make sure it runs in the project / App. We use the `metadata` to namespace and group the objects we create.

If we think of the entire yaml document as a constructor the `spec` In the case of this service, it needs to be running on port 80 and load balance between deployments (which we will create later) with a specific label

<pre class="file" data-filename="wordpress-config.yml" data-target="$">
---
apiVersion: v1
kind: Service
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  ports:
    - port: 80
  selector:
    app: wordpress
    tier: frontend
  type: LoadBalancer
</pre>

After adding the definition of the Service to your `wordpress-config.yml` we will want to create the object. We can do that by running the folowing `kubectl` command.

`kubectl apply -f wordpress-config.yml`{{execute}}

We should see a success message afterward indicating the object was created. Lets inspect our cluster and see that it created our object. 

`kubectl get Service`{{exeucte}}

We should see two services listed. If we want to narrow this down, we can just get the `app` labled service.

`kubectl get Service -l app`{{execute}}

