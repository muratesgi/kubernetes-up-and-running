Running Pods

$

Listing Pods

$ kubectl get pods

Pod Details

$ kubectl describe pods kuard

Deleting a Pod

When it is time to delete a Pod, you can delete it either by name:

$ kubectl delete pods/kuard

or using the same file that you used to create it:

$ kubectl delete -f kuard-pod.yaml

Accessing Your Pod

$ kubectl port-forward kuard 8080:8080

> > > http://localhost:8080

Getting More Info with Logs

$ kubectl logs kuard

Adding the -f flag will cause you to continuously stream logs.
The kubectl logs command always tries to get logs from the currently running container.
Adding the --previous flag will get logs from a previous instance of the container.
This is useful, for example, if your containers are continuously restarting due
to a problem at container startup.

Running Commands in Your Container with exec

Sometimes logs are insufficient, and to truly determine what’s going on you need to
execute commands in the context of the container itself. To do this you can use:

$ kubectl exec kuard date

You can also get an interactive session by adding the -it flags:

$ kubectl exec -it kuard ash

Health Checks

When you run your application as a container in Kubernetes, it is automatically kept
alive for you using a process health check. This health check simply ensures that the
main process of your application is always running. If it isn’t, Kubernetes restarts it.
However, in most cases, a simple process check is insufficient. For example, if your
process has deadlocked and is unable to serve requests, a process health check will
still believe that your application is healthy since its process is still running.
To address this, Kubernetes introduced health checks for application liveness.
Liveness health checks run application-specific logic (e.g., loading a web page) to verify
that the application is not just still running, but is functioning properly. Since
these liveness health checks are application-specific, you have to define them in your
Pod manifest.

Liveness Probe

Once the kuard process is up and running, we need a way to confirm that it is
actually healthy and shouldn’t be restarted. Liveness probes are defined per container,
which means each container inside a Pod is health-checked separately.
In kuard-pod-health.yaml we add a liveness probe to our kuard container, which runs an HTTP
request against the /healthy path on our container.

The preceding Pod manifest uses an httpGet probe to perform an HTTP GET request
against the /healthy endpoint on port 8080 of the kuard container. The probe sets an
initialDelaySeconds of 5, and thus will not be called until 5 seconds after all the
containers in the Pod are created. The probe must respond within the 1-second timeout,
and the HTTP status code must be equal to or greater than 200 and less than 400
to be considered successful. Kubernetes will call the probe every 10 seconds. If more
than three consecutive probes fail, the container will fail and restart.

You can see this in action by looking at the kuard status page. Create a Pod using this
manifest and then port-forward to that Pod:

$ kubectl apply -f kuard-pod-health.yaml

$ kubectl port-forward kuard 8080:8080

Point your browser to http://localhost:8080. Click the “Liveness Probe” tab. You
should see a table that lists all of the probes that this instance of kuard has received. If
you click the “Fail” link on that page, kuard will start to fail health checks. Wait long
enough and Kubernetes will restart the container. At that point the display will reset
and start over again. Details of the restart can be found with kubectl describe pods
kuard. The “Events” section will have text similar to the following:

Killing container with id docker://2ac946...:pod "kuard_default(9ee84...)"
container "kuard" is unhealthy, it will be killed and re-created.

Resource Requests: Minimum Required Resources

With Kubernetes, a Pod requests the resources required to run its containers. Kubernetes
guarantees that these resources are available to the Pod. The most commonly requested resources are CPU and memory, 
but Kubernetes has support for other resource types as well, such as GPUs and more.

>>>kuard-pod-resreq.yaml

Resources are requested per container, not per Pod. The total
resources requested by the Pod is the sum of all resources requested
by all containers in the Pod. The reason for this is that in many
cases the different containers have very different CPU requirements.
For example, in the web server and data synchronizer Pod,
the web server is user-facing and likely needs a great deal of CPU,
while the data synchronizer can make do with very little.

Capping Resource Usage with Limits
In addition to setting the resources required by a Pod, which establishes the minimum
resources available to the Pod, you can also set a maximum on a Pod’s resource
usage via resource limits.
In our previous example we created a kuard Pod that requested a minimum of 0.5 of
a core and 128 MB of memory. In the Pod manifest in kuard-pod-reslim.yaml, we extend this
configuration to add a limit of 1.0 CPU and 256 MB of memory.

Persisting Data with Volumes

When a Pod is deleted or a container restarts, any and all data in the container’s filesystem
is also deleted. This is often a good thing, since you don’t want to leave around
cruft that happened to be written by your stateless web application. In other cases,
having access to persistent disk storage is an important part of a healthy application.
Kubernetes models such persistent storage

Using Volumes with Pods

To add a volume to a Pod manifest, there are two new stanzas to add to our configuration.
The first is a new spec.volumes section. This array defines all of the volumes
that may be accessed by containers in the Pod manifest. It’s important to note that not
all containers are required to mount all volumes defined in the Pod. The second addition
is the volumeMounts array in the container definition. This array defines the volumes
that are mounted into a particular container, and the path where each volume
should be mounted. Note that two different containers in a Pod can mount the same
volume at different mount paths.

>>>kuard-pod-vol.yaml

Putting It All Together
Many applications are stateful, and as such we must preserve any data and ensure
access to the underlying storage volume regardless of what machine the application
runs on. As we saw earlier, this can be achieved using a persistent volume backed by
network-attached storage. We also want to ensure that a healthy instance of the
application is running at all times, which means we want to make sure the container
running kuard is ready before we expose it to clients.
Through a combination of persistent volumes, readiness and liveness probes, and
resource restrictions, Kubernetes provides everything needed to run stateful applications
reliably. kuard-pod-full.yaml pulls this all together into one manifest.

>>>kuard-pod-full.yaml
