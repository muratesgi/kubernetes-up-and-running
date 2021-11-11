Service Discovery

Service-discovery
tools help solve the problem of finding which processes are listening at
which addresses for which services. A good service-discovery system will enable users
to resolve this information quickly and reliably. A good system is also low-latency;
clients are updated soon after the information associated with a service changes.
Finally, a good service-discovery system can store a richer definition of what that service
is. For example, perhaps there are multiple ports associated with the service.
The Domain Name System (DNS) is the traditional system of service discovery on
the internet. DNS is designed for relatively stable name resolution with wide and efficient
caching. It is a great system for the internet but falls short in the dynamic world
of Kubernetes.

The Service Object

Real service discovery in Kubernetes starts with a Service object.

Just as the kubectl run command is an easy way to create a Kubernetes deployment,
we can use kubectl expose to create a service. Let’s create some deployments and
services so we can see how they work:
---
$ kubectl run alpaca-prod \
--image=gcr.io/kuar-demo/kuard-amd64:blue \
--replicas=3 \
--port=8080 \
--labels="ver=1,app=alpaca,env=prod"
$ kubectl expose deployment alpaca-prod
$ kubectl run bandicoot-prod \
--image=gcr.io/kuar-demo/kuard-amd64:green \
--replicas=2 \
--port=8080 \
--labels="ver=2,app=bandicoot,env=prod"
$ kubectl expose deployment bandicoot-prod
$ kubectl get services -o wide

---