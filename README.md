## EKS Cost Optimization Approach — Automated Scaling with K8s CronJobs

This cost savings solution uses Kubernetes CronJobs to automatically scale down all deployments across namespaces used to host applications during off-peak hours and weekend, and it scales them back up during peak hours.
The solution is expected to reduce compute cost on our EKS Managed Node Group and Karpenter Worker Nodes respectively. In addition, it helps to reduce resource consumption and cloud costs in non-production environment.

## How it works 

```
CronJob	Schedule         (EST)	   Days	        Action
scale-up-business-hours	06:00	     Mon–Fri	    Scale all deployments to desired replicas
scale-down-after-hours	20:00	     Mon–Fri	    Scale all deployments to 0
scale-down-weekends	    hourly	   Sat–Sun	    Enforce 0 replicas across all namespaces
```
The weekend CronJob runs hourly to guard against any accidental manual scale-ups during the weekend.

## Repository Structure
```
.
├── rbac.yaml                          # ServiceAccount, ClusterRole, ClusterRoleBinding
├── cronjob-scale-up-weekdays.yaml     # Scale up Mon–Fri at 06:00 EST
├── cronjob-scale-down-weekdays.yaml   # Scale down Mon–Fri at 20:00 EST
└── cronjob-scale-down-weekends.yaml   # Enforce zero replicas Sat–Sun

```

## RBAC

All CronJobs use a single ServiceAccount named deployment-scaler created in the kube-system namespace. A ClusterRole and ClusterRoleBinding grant it the minimum permissions needed to operate across all namespaces.

The ServiceAccount lives in kube-system because it performs cluster-wide operations not tied to any single application namespace. The CronJobs also run in kube-system so the pod spec can reference the ServiceAccount directly (Kubernetes requires the pod and its ServiceAccount to be in the same namespace).

# Namespace Filtering
The scripts automatically exclude Kubernetes system namespaces from scaling cluster level software addons:

- kube-system
- kube-public
- kube-node-lease
