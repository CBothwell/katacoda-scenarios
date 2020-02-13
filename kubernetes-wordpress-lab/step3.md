# Add a Secret

Secrets are supposed to be that, secret. But we're going to play it a little fast and loose today since we want to get the concept down and not nessisarily get to the best practises yet. 

You can learn more about how to use [secrets](https://kubernetes.io/docs/concepts/configuration/secret/) at the kubernetes documentation on them. 

We're only going to need the one secret, `MYSQL_ROOT_PASSWORD`. First we're going to need to generate the base64 encoded value of a password.

`printf '%s' $(cat /dev/urandom | tr -dc 'a-zA-Z0-9' | fold -w 16 | head -n 1) | base64`{{execute}}

You'll want to copy this password for later.

Lets add the secret to our config.

<pre class="file" data-filename="wordpress-config.yml" data-target="append">
---
apiVersion: v1
kind: Secret
metadata:
  name: mysql-pass
  labels:
    app: wordpress
type: Opaque
data:
  password: $COPIED_PASSWORD
</pre>

Lets replace `$COPIED_PASSWORD` with the generated password we copied from the CLI. 

You may note we don't have a `spec` but instead have a `data`. Secrets contain key value pairs defined in `data` with the values being base64 encoded.

Now lets create the secret.

`kubectl apply -f wordpress-config.yml`{{execute}}

And lets confirm it was created. 

`kubectl get secret/mysql-pass`{{execute}}