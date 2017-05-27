# Kubernetes Dashboard Ingress w/ Basic Auth
Note:  These instructions are but a slightly more specific version of [these instructions here](https://docs.giantswarm.io/guides/advanced-ingress-configuration/#authentication).

So you wanna' hit up the [Kubernetes Dashboard](https://github.com/kubernetes/dashboard)
running on your cluster via an external URL but you wanna' keep that thing locked down with a username and password.
Smart thinking!  [This is how we do it.](https://www.youtube.com/watch?v=0hiUuL5uTKc)  Start by cloning this project
to a directory in a location accessible to your `kubectl` and then `cd` into that directory.

### Create an `auth` file with encrypted username & password
Start by creating a file called `auth` containing your desired username and password using the `htpasswd` utility:

```bash
$ htpasswd -c auth <your username here>
New password: <your password here>
Re-type new password: <your password here again>
Adding password for user <username>
```

`cat` the newly created `auth` file to see what `htpasswd` created.  C'mon... you know you wanna' see what's in there.

If you want to add additional usernames/passwords to the `auth` file, simply rerun `htpasswd` without the `-c` argument:

```bash
$ htpasswd auth <new username>
New password: <new password>
Re-type new password: <new password again.
Adding password for user <new username>
```

### Create a secret from the `auth` file
Next, you'll need to create a secret called `dashboard-auth-secret` in the `kube-system` namespace.  The ingress we're
about to create will look for a secret called `dashboard-auth-secret` which is why this secret needs to be named thusly.

```bash
$ kubectl create secret generic dashboard-auth-secret --from-file=auth -n kube-system
```

Run this command to see what Kubernetes thinks of your dirty little secret:

```bash
$ kubectl get secret dashboard-auth-secret -o yaml -n kube-system
```

### Deploy the Kubernetes Web Ui (Dashboard)
Deploy the Kubernetes Dashboard if haven't already.  Instructions here: (https://github.com/kubernetes/dashboard)
The various components of the dashboard run in the `kube-system` namespace which is why we need to create our secret 
and ingress in that same namespace as well.

### Deploy the Dashboard Ingresss
Now that you've got your secret in place and the dashboard is up and running and has presumably created a
service called `kubernetes-dashboard`, let's create that frickin' ingress already!  You'll need to modify
the value of the `host` setting in `dashboard-ingress.yaml` to match the URL that you want the dashboard to
be reachable at.  After you've done that, simply run:

```bash
$ kubectl apply -f dashboard-ingress.yaml 
```

That ought to do it.  Your ingress should be in place and routing traffic from the dashboard URL that you specified 
in the `dashboard-ingress.yaml` file to your Kubernetes dashboard *and* you should be prompted for a username and
password when you try to hit it.

### Tearing it down
If you've got a hankerin' to tear down all that you've just created, the following commands ought to do it:

```bash
$ kubectl delete -f dashboard-ingress.yaml
$ kubectl delete secret dashboard-auth-secret -n kube-system
```

Assuming that you deployed the dashboard by following [these instructions](https://github.com/kubernetes/dashboard),
you can delete the dashboard deployment like so:

```bash
$ kubectl delete -f https://git.io/kube-dashboard
```
