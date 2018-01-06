# Local development with Docker for Mac and Kubernetes

Docker for Mac's [new Kubernetes support](https://docs.docker.com/docker-for-mac/#kubernetes) enables some pretty rad patterns for local development. If you run services on Kubernetes in production are are interested in running them in a similar environment on your Mac, you might be interested in some of these patterns.

## Accessing services from your Mac

Docker for Mac exposes Kubernetes Services of type `LoadBalancer` to your Mac's network using `vpnkit`. There's an example echo server in [`./echo/`](./echo/). Here's how it works on my Mac:

> Apply the resources in [`./echo/`](./echo/) to start the echo server:

```
$ kubectl apply -Rf ./echo/
deployment "echo" configured
service "echo" configured
```

> Confirm the deployment is running:

```
$ kubectl get deployment echo
NAME      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
echo      1         1         1            1           1m
```

> Test it out:

```
$ sudo lsof -Pnitcp | grep 3333
vpnkit    54100 jnewland   32u  IPv4 0x86c54c0d101bc03f      0t0  TCP *:3333 (LISTEN)
vpnkit    54100 jnewland   34u  IPv6 0x86c54c0cee4cef17      0t0  TCP [::1]:3333 (LISTEN)

$ echo "$RANDOM" | nc localhost 3333
144
```

> Cleanup:

```
$ kubectl delete svc echo
service "echo" deleted
$ kubectl delete deployment echo
deployment "echo" deleted
```

There are only so many ports, though, and it's kinda hard to remember what port is assigned to an application you rarely use. If you expect to run multiple Kubernetes applications on your Mac (say, a _business_ of microservices), you might want to be able to access each of these services without remembering which port you've allocated to them.

### Accessing HTTP(s) services with nginx-ingress and xip.io

The [NGINX Ingress Controller](https://github.com/kubernetes/ingress-nginx) can run on ports 80 and 443, allowing you to configure which Service is available at a given hostname using [Ingress resources](https://kubernetes.io/docs/concepts/services-networking/ingress/). Combined with [xip.io](http://xip.io), you can configure access to HTTP services locally using addresses like `http://$service.127.0.0.1.xip.io/`.

#### Setup nginx-ingress

* Disable other things listening on port 80 / 443 on your Mac:
  * (You might need to `sudo brew services stop launch_socket_server`)
* Install a recent version of Docker for Mac and [enable Kubernetes](https://docs.docker.com/docker-for-mac/#kubernetes)
* Deploy nginx-ingress:

```
kubectl apply -f nginx-ingress/namespaces/nginx-ingress.yaml -Rf nginx-ingress
```

* Visit http://127.0.0.1.xip.io/, which should show an unstyled 404.

#### Setup the example application

[`./httpbin/`](./httpbin/) contains an example a the configuration that might live in an individual application repo. If the all members of your engineering organization have a similar ingress controller running on their development machines, the development patterns in that directory can be used for multiple services. That's pretty cool.
