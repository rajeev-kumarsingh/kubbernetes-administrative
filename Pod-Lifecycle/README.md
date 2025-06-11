# Kubernetes Pod Lifecycle

## 1. Overview

A **Pod** in Kubernetes goes through a defined set of phases during its lifetime. These phases help understand the current state of the Pod and determine whether it is running, terminating, or failed.

Kubernetes also emits **events** and **status conditions** to provide more detail about what's happening with a Pod.

---

## 2. Pod Phases

### 1. `Pending`

- The Pod has been accepted by the Kubernetes system, but one or more of the containers are not yet running.
- Reasons may include: image pull delay, scheduling delay, or waiting for volumes.

### 2. `Running`

- The Pod has been bound to a node, and all containers have been created.
- At least one container is running or is in the process of starting or restarting.

### 3. `Succeeded`

- All containers in the Pod have completed successfully and will not restart.

### 4. `Failed`

- All containers in the Pod have terminated, and at least one container terminated in failure (non-zero exit code).

### 5. `Unknown`

- The state of the Pod could not be obtained, usually due to communication errors with the node.

---

## 3. Pod Lifecycle Events

### Important events that influence Pod behavior:

- **Scheduled**: The Pod is scheduled to a Node.
- **Pulled**: Container image pulled.
- **Created**: Container created.
- **Started**: Container started.
- **Killing**: Pod is terminating.
- **BackOff**: Container is crashing repeatedly.

You can view events using:

```bash
kubectl describe pod <pod-name>

```

### 4. Example: YAML with Init Container and Lifecycle Hook

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: lifecycle-pod
spec:
  initContainers:
    - name: init-myservice
      image: busybox
      command: ["sh", "-c", "echo Initializing...; sleep 5"]
  containers:
    - name: app-container
      image: busybox
      command: ["sh", "-c", "echo App running...; sleep 3600"]
      lifecycle:
        postStart:
          exec:
            command: ["/bin/sh", "-c", "echo Starting up..."]
        preStop:
          exec:
            command: ["/bin/sh", "-c", "echo Shutting down..."]
```

This Pod has:

- An init container for initialization logic.

- A main container with postStart and preStop hooks to manage lifecycle.

---

## 5. Interview Questions and Answers

Q1: What are the different Pod phases in Kubernetes?

A: Pending, Running, Succeeded, Failed, and Unknown.

Q2: What happens when a Pod is in Pending state?

A: The Pod has been created but not yet scheduled to a node or is waiting for resources like images or volumes.

Q3: What does the Succeeded phase indicate?

A: All containers have terminated successfully and the Pod will not be restarted.

Q4: Can a Pod move from Running to Pending?

A: No, phase transitions do not work that way. Once a Pod is in Running, it can only move to Succeeded or Failed.

Q5: What is the role of initContainers in Pod lifecycle?

A: initContainers run to completion before main containers start. They are used for initialization logic.

Q6: How is a Pod terminated gracefully?

A: Kubernetes sends a TERM signal to containers, invokes preStop hooks, waits for the grace period (default 30s), and then sends a KILL signal if the container hasn’t exited.

Q7: How can you debug a Pod stuck in Pending?

A: Use kubectl describe pod <pod-name> to check for events like volume attachment, image pull issues, or scheduling problems.

Q8: What is a Pod condition?

A: Conditions are granular status fields like Ready, Initialized, etc., that describe the state of the Pod more specifically than the phase.
