# Common Kubernetes Real Time Challenges 

## 1. Resource Sharing

**Problem:** 
- When multiple teams or services use the same Kubernetes cluster, resource contention can occur. 
- If one team’s application consumes excessive resources (e.g., due to memory leaks or insufficient testing), it can affect the entire cluster, leading to:
    - Other teams' applications crashing due to insufficient resources.
    - Pods going into "CrashLoopBackOff" with "OOMKilled" (Out of Memory Killed) errors.
- Example: A cluster with 100 CPUs and 100 GB RAM is shared among namespaces. If the payments team’s namespace has a pod that leaks memory and consumes 32 GB RAM instead of 6 GB, it could deprive other namespaces of resources.

**Solution:**
- *Resource Quotas:*
    - Set limits on the amount of CPU and memory each namespace can consume. For instance, allocate 15 GB RAM and 15 CPUs to one namespace, ensuring it doesn't exceed its share.
    - Resource quotas reduce the "blast radius" of issues caused by a single namespace consuming too many resources.
    - With resource quotas, resource misuse in one namespace doesn't affect the entire cluster.
- *Resource Limits:*
    - Define the maximum CPU and memory that a pod can consume.
    - This prevents one pod from monopolizing namespace resources, reducing the impact to only the misbehaving pod.
    - With resource limits, issues are confined to the specific pod, enabling easier debugging and minimizing disruption.

## 2. Handling OOM (Out of Memory) Errors

**Problem:** 
- Even after setting resource quotas and limits, a pod might still crash due to memory issues.
- "OOMKilled" occurs when a pod exceeds its memory limit. This results in:
    - Pod termination.
    - A "CrashLoopBackOff" status.
- Example: A Java application in a pod has a memory leak. Even though the pod is limited to 8 GB RAM, it consumes more memory, triggering the OOM killer.

**Solution:**
- Identify the pod in a "CrashLoopBackOff" state using `kubectl get pods`.
- Diagnose the issue with `kubectl describe pod <pod-name>` to confirm the OOMKilled error.
- Check the pod’s status and reason for failure with `kubectl describe pod <pod-name>`.
    - Look for `Reason: OOMKilled` in the events section.
- If it's a Java application (as an example), retrieve diagnostic data:
    - *Heap Dump:* Captures memory allocation at the time of failure. Use tools like `jmap` or Java Profilers to generate the dump.
    - *Thread Dump:* Shows running threads and their states. Use `jstack` or send a `kill -3` signal to the process.
- Share these diagnostic dumps with the development team for analysis.
    - Developers analyze the dumps to find the root cause of the memory issue, fix the code, and deploy a new version.

## 3. Upgrades

**Problem:** 
- Keeping Kubernetes clusters updated (e.g., from version 1.29 to 1.30) can be challenging due to:
    - Breaking changes in new versions.
    - Risk of downtime during the upgrade.

**Solution:**
- *Preparation:*
    - Read release notes to identify deprecated features and changes.
    - Plan for compatibility.
    - Backup the cluster configuration and resources.
- *Step-by-Step Upgrade:*
    - Start with control plane components (e.g., `etcd`, API server, scheduler).
        - Upgrade etcd, API server, controller manager, and scheduler in sequence
    - Upgrade worker nodes in a rolling manner:
        - Drain one node: Migrate pods to other nodes.
        - Upgrade Kubernetes components on the node.
        - Bring the node back online and repeat for other nodes.
- *Testing:*
    - Validate that all workloads function as expected after the upgrade.
- *Documentation:*
    - Prepare detailed manuals for future upgrades, including steps, precautions, and rollback plans.

