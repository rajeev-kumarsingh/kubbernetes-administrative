# Kubernetes Container Lifecycle Hooks

## 1. Overview

Container lifecycle hooks in Kubernetes allow you to execute custom logic when a container starts (`postStart`) and before it stops (`preStop`). These hooks are useful for initialization, cleanup, or graceful shutdown tasks.

---

## 2. Lifecycle Hook Types

### `postStart`

- Triggered immediately after the container starts.
- Executes custom initialization logic.
- If it fails, the container is killed and restarted.

### `preStop`

- Triggered before the container is terminated.
- Used for cleanup, draining traffic, or graceful shutdown.
- Has 30 seconds (default) to complete before the container is forcibly killed.

---

## 3. YAML Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: lifecycle-demo
spec:
  containers:
    - name: busybox
      image: busybox
      args: ["/bin/sh", "-c", "while true; do echo Running...; sleep 5; done"]
      lifecycle:
        postStart:
          exec:
            command:
              [
                "/bin/sh",
                "-c",
                "echo Hello from the postStart handler > /var/log/poststart.log",
              ]
        preStop:
          exec:
            command:
              [
                "/bin/sh",
                "-c",
                "echo Goodbye from the preStop handler > /var/log/prestop.log",
              ]
```

# 4. Interview Questions and Answers

Q1: What are container lifecycle hooks in Kubernetes?

Ans: Lifecycle hooks let you run custom logic during a container's startup (postStart) and shutdown (preStop).

Q2: What is the difference between postStart and preStop hooks?

A: postStart runs after the container starts. preStop runs before it's terminated.

Q3: What happens if the postStart hook fails?

A: The container is considered failed and will be restarted based on its restart policy.

Q4: How much time does preStop hook get to complete?

A: It gets the full terminationGracePeriodSeconds (default 30 seconds).

Q5: Can you use HTTP or TCP checks in lifecycle hooks?

A: No, lifecycle hooks only support exec commands, not HTTP or TCP.

Q6: What are some use cases for preStop hook?

A: Graceful shutdown, draining traffic, deregistering from a load balancer, etc.

Q7: Are lifecycle hooks blocking?

A: Yes, postStart is blocking in the sense that it runs after the container starts but doesn't delay readiness. preStop is blocking during shutdown.

Q8: Can we use lifecycle hooks in Deployments?

A: Yes, hooks can be defined inside container specs in any workload object (e.g., Deployment, StatefulSet, Pod).
