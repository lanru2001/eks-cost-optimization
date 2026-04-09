# Kubernetes Dev Cluster — Automated Scaling with K8s CronJobs

This cost savings solution uses Kubernetes CronJobs to automatically scale down all deployments across namespaces used to host applications during off-peak hours and weekend, and it scales them back up during peak hours.
The solution is expected to reduce compute cost on our EKS Managed Node Group and Karpenter Worker Nodes respectively. In addition, it helps to reduce resource consumption and cloud costs in non-production environment.

# How it works 

```
CronJob	Schedule         (EST)	   Days	        Action
scale-up-business-hours	06:00	     Mon–Fri	    Scale all deployments to desired replicas
scale-down-after-hours	20:00	     Mon–Fri	    Scale all deployments to 0
scale-down-weekends	    hourly	   Sat–Sun	    Enforce 0 replicas across all namespaces
```
The weekend CronJob runs hourly to guard against any accidental manual scale-ups during the weekend.
