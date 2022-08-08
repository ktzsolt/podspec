# Kubernetes Pod Specification Good Defaults

The Pod spec for your apps can be one of the more complex parts of your Kubernetes manifest design, and needs many features enabled to be a save and reasonably secure default.

This single-file repository is meant to be a starting point for your Pod specs, to add to Deployments, DaemonSets, StatefulSets, initContainers, etc.

```yaml
# generic pod spec that's usable inside a deployment or other higher level k8s spec
# via https://github.com/BretFisher/podspec

apiVersion: v1
kind: Pod
metadata:
  name: mypod

spec:

  containers:

      # basic container details
    - name: my-container-name
      # never use reusable tags like latest or stable
      image: my-image:tag
      # hardcode the listening port if Dockerfile isn't set with EXPOSE
      ports:
        - containerPort: 8080
          protocol: TCP

      readinessProbe:        # only needed if your pod has a service and listening port
        httpGet:             # Lots of timeout values with defaults, be sure they are ideal for your workload
          path: /ready
          port: 8080
      livenessProbe:         # only needed if your app becomes unresponsive or you don't have a readinessProbe, but this is up for debate.
        httpGet:             # Lots of timeout values with defaults, be sure they are ideal for your workload
          path: /ready
          port: 8080

      resources:             # Because if limits = requests then QoS is set to "Guaranteed"
        limits:
          memory: "500Mi"    # If container uses over 500MB it is killed (OOM)
          cpu: "2"           # If container uses over 2 vCPU it is throttled
        requests:
          memory: "500Mi"    # Scheduler finds a node where 500MB is available
          cpu: "1"           # Scheduler finds a node where 1 vCPU is available

      # per-container security context
      # lock down privileges inside the container
      securityContext:
        allowPrivilegeEscalation: false # prevent sudo, etc.
        privileged: false               # prevent acting like host root

  # per-pod security context
  # enable seccomp and force non-root user
  securityContext:

    seccompProfile:
      type: RuntimeDefault   # enable seccomp and the runtimes default profile

    runAsUser: 1001          # hardcode user to non-root if not set in Dockerfile
    runAsGroup: 1001         # hardcode group to non-root if not set in Dockerfile
    runAsNonRoot: true       # hardcode to non-root. Redundant to above if Dockerfile is set USER 1000
```
