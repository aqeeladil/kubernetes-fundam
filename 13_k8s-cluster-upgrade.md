# Kubernetes Cluster Upgrades

## Why Upgrade Kubernetes?

- Kubernetes releases new versions approximately every 3 months.
- Only the latest three versions are supported (e.g., if 1.32 is the latest, only 1.32, 1.31, and 1.30 are supported).
- Keeping Kubernetes clusters up-to-date is critical for stability, security, and access to the latest features.

## Prerequisites for Kubernetes Upgrades

1. **Cordon Nodes (Optional but Recommended):**
    - Cordon the nodes (mark them unschedulable) to prevent new workloads from being scheduled during the upgrade.
    - This ensures stability during the upgrade, especially in production-critical environments.
    - `kubectl cordon <node-name>`
    - This makes the node unschedulable but does not disrupt existing workloads.

2. **Review Kubernetes Release Notes:**
    - Kubernetes versions introduce changes such as:
        - API Deprecations: Old APIs might be removed or replaced (e.g., Ingress API moved from `extensions/v1beta1` to `networking.k8s.io/v1`).
        - Feature Changes: Behavior of existing features might change, impacting configurations.
    - Carefully read the release notes for the target version to identify breaking changes and prepare accordingly.

3. **Test in Lower Environments:**
    - Always test upgrades in staging, pre-production, or other lower environments before upgrading production.
    - Allow at least 1–2 weeks to observe and validate the stability of the upgraded environments.
    - Key Point: Kubernetes upgrades are not reversible. If something breaks in production, you must reinstall the cluster from scratch to roll back.
4. **Version Synchronization:**
    - Ensure the control plane and nodes are running the same Kubernetes version.
    - Nodes cannot skip intermediate versions. For example:
        - If upgrading to 1.31, ensure nodes are first upgraded from 1.29 to 1.30, and only then proceed to 1.31.
5. **Cluster Autoscaler Compatibility:**
    - If you use Kubernetes Cluster Autoscaler, ensure the autoscaler version is compatible with the upgraded control plane version.
    - An incompatible autoscaler can lead to issues with pod scaling and cluster behavior.
6. **IP Address Availability:**
    - Ensure there are at least five available IP addresses in the subnet where the cluster resides.
    - These are used during the upgrade process for creating temporary resources.
7. **Synchronize kubectl Version:**
    - Ensure the kubectl version matches the upgraded control plane version.
    - `kubectl version --short`

## Steps to Upgrade a Kubernetes Cluster
The upgrade process is divided into three key phases:

### Step 1: Upgrade the Control Plane
- The control plane manages the cluster's API server, etcd database, and other core services.
    - If using a managed service like AWS EKS, run a command such as:
    - `eksctl upgrade cluster --name <cluster-name> --region <region> --version <new-version> --approve`
- The control plane upgrade typically takes 25–30 minutes.
- AWS manages the control plane's high availability, disaster recovery, and scalability, but the upgrade must still be triggered manually.

### Step 2: Upgrade the Nodes (Data Plane)
- Nodes run your application workloads, and their Kubernetes version must match the control plane.

Options for upgrading nodes (Managed Node Groups are recommended for simpler upgrades):
- **Managed Node Groups:**
    - Use the AWS console or CLI to initiate a rolling update.
    - Nodes are updated sequentially to ensure zero downtime.
    - Steps:
        - Go to the AWS Console → EKS → Compute → Node Groups.
        - Select the node group and click Update Now.
        - Use the Rolling Update strategy to sequentially update nodes.
- **Self-Managed Nodes:**
    - Upgrade nodes individually using custom AMIs or configurations.
    - If using custom AMIs or launch templates, you'll need to update these manually.
    - Cordon and drain each node, upgrade it, and then reintroduce it to the cluster.
    - Steps:
        - Cordon and Drain the Node:
            - `kubectl cordon <node-name>`
            - `kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data`
        - Update the node’s Kubernetes version or replace it with a new node.
        - Rejoin the node to the cluster.
- **Add A New Node Groups:**
    - Create a new node group with the upgraded version, migrate workloads, and remove the old group.
- **Managed vs. Self-Managed Nodes:**
    - Managed Node Groups: Easier to upgrade; AWS handles the process.
    - Self-Managed Nodes: Requires manual effort but allows greater customization.

### Step 3: Upgrade Kubernetes Add-ons
- Update any installed add-ons, such as:
    - VPC CNI
    - kube-proxy
    - CoreDNS
- This step ensures compatibility with the new Kubernetes version.
- Steps:
    - Check for available updates in the AWS Console.
    - Click Update to install the latest compatible version.
    - Ensure add-ons are compatible with the new control plane version.

## Post-Upgrade Verification
1. **Run Functional Tests:**
    - Run end-to-end (E2E) or regression tests to ensure that applications and workloads are functioning correctly.
    - Example: Check if deployments, services, and ingress are functioning correctly.
    - Focus on functional testing rather than individual components like Helm, ArgoCD, or Prometheus.
2. **Check Add-ons:**
    - Verify that Helm, ArgoCD, Prometheus, and other controllers are functioning correctly.
    - If possible, automate validation testing. Automating tests for the upgraded cluster ensures faster and more reliable validation.
3. **Review Logs:**
    - Use kubectl logs and monitoring tools to check for errors or warnings after the upgrade.

## Key Considerations for Zero Downtime
- Use Rolling Updates to update nodes sequentially, minimizing impact on workloads.
- Avoid Forceful Updates unless Pod Disruption Budgets (PDBs) are in place and carefully configured.
- Ensure lower environments mirror production for accurate testing.
- Perform upgrades during maintenance windows and notify teams in advance.


