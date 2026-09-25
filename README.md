# Imperative Commands for CKA

A practical, exam-focused reference for Kubernetes imperative commands you can type quickly during the CKA exam and in real cluster troubleshooting.

> **Tip:** Prefer `--dry-run=client -o yaml` when you need to generate a manifest instead of creating the resource immediately.

## Quick command patterns

| Purpose | Pattern |
|---|---|
| Preview without creating | `--dry-run=client` |
| Generate YAML | `-o yaml` |
| Save YAML to a file | `> resource.yaml` |
| Create from YAML | `kubectl apply -f resource.yaml` |
| Inspect generated YAML | `kubectl ... --dry-run=client -o yaml` |

## 1. Pods

### Create a Pod
```bash
kubectl run nginx --image=nginx
```

### Generate a Pod manifest
```bash
kubectl run nginx --image=nginx --dry-run=client -o yaml
```

### Generate and save the manifest
```bash
kubectl run nginx --image=nginx --dry-run=client -o yaml > nginx-pod.yaml
```

### Pod with a command
```bash
kubectl run nginx --image=nginx --command -- sleep 3600
```

### Pod with a label
```bash
kubectl run nginx --image=nginx --labels=app=web
```

### Pod with an environment variable
```bash
kubectl run nginx --image=nginx --env="APP_ENV=prod"
```

## 2. Deployments

### Create a Deployment
```bash
kubectl create deployment nginx --image=nginx
```

### Generate a Deployment manifest
```bash
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml
```

### Create a Deployment with replicas
```bash
kubectl create deployment nginx --image=nginx --replicas=4
```

### Scale a Deployment
```bash
kubectl scale deployment nginx --replicas=4
```

### Update the container image
```bash
kubectl set image deployment/nginx nginx=nginx:1.27
```

## 3. Services

### Expose a Pod as ClusterIP
```bash
kubectl expose pod redis --port=6379 --name=redis-service --dry-run=client -o yaml
```

This approach derives the Service selector from the Pod labels.

### Generate a ClusterIP Service
```bash
kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml
```

Check the generated selectors before applying when the Pod labels are not the default `app=<name>` pattern.

### Expose a Pod as NodePort
```bash
kubectl expose pod nginx --type=NodePort --port=80 --name=nginx-service --dry-run=client -o yaml
```

To control the exact nodePort, generate the YAML, add `spec.ports[].nodePort`, then apply it.

### Generate a NodePort Service with an explicit node port
```bash
kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml
```

## 4. Namespaces and Context

### Create a namespace
```bash
kubectl create namespace dev
```

### Generate a namespace manifest
```bash
kubectl create namespace dev --dry-run=client -o yaml
```

### Run a command in a namespace
```bash
kubectl -n dev get pods
```

### List contexts
```bash
kubectl config get-contexts
```

### Switch context
```bash
kubectl config use-context <context-name>
```

### Set the current context namespace
```bash
kubectl config set-context --current --namespace=dev
```

## 5. ConfigMaps

### Create from literals
```bash
kubectl create configmap app-config --from-literal=ENV=prod --from-literal=LOG_LEVEL=info
```

### Create from a file
```bash
kubectl create configmap app-config --from-file=app.properties
```

### Generate YAML
```bash
kubectl create configmap app-config --from-literal=ENV=prod --dry-run=client -o yaml
```

## 6. Secrets

### Create a generic Secret
```bash
kubectl create secret generic db-secret --from-literal=username=admin --from-literal=password='change-me'
```

### Generate a Secret manifest
```bash
kubectl create secret generic db-secret --from-literal=username=admin --dry-run=client -o yaml
```

### Create a Docker registry Secret
```bash
kubectl create secret docker-registry regcred \
  --docker-server=<registry> \
  --docker-username=<username> \
  --docker-password=<password> \
  --docker-email=<email>
```

> Never commit real credentials to a public repository.

## 7. Jobs

### Create a Job
```bash
kubectl create job hello --image=busybox -- echo "hello"
```

### Generate a Job manifest
```bash
kubectl create job hello --image=busybox --dry-run=client -o yaml -- echo "hello"
```

### Inspect a Job
```bash
kubectl get jobs
kubectl describe job hello
```

## 8. CronJobs

### Create a CronJob
```bash
kubectl create cronjob hello --image=busybox --schedule="*/5 * * * *" -- echo "hello"
```

### Generate a CronJob manifest
```bash
kubectl create cronjob hello --image=busybox --schedule="*/5 * * * *" --dry-run=client -o yaml -- echo "hello"
```

## 9. Troubleshooting

### Get resources
```bash
kubectl get pods
kubectl get pods -A
kubectl get all -n dev
```

### Wide output
```bash
kubectl get pods -o wide
```

### Sort Pods by node
```bash
kubectl get pods -A -o wide --sort-by=.spec.nodeName
```

### Describe resources
```bash
kubectl describe pod <pod>
kubectl describe node <node>
```

### Logs
```bash
kubectl logs <pod>
kubectl logs <pod> -c <container>
kubectl logs <pod> --previous
kubectl logs -f <pod>
```

### Execute commands
```bash
kubectl exec -it <pod> -- sh
kubectl exec -it <pod> -c <container> -- sh
```

### Port-forward
```bash
kubectl port-forward pod/<pod> 8080:80
kubectl port-forward svc/<service> 8080:80
```

## 10. Editing and Patching

### Edit a resource
```bash
kubectl edit deployment nginx
```

### Patch a resource
```bash
kubectl patch deployment nginx -p '{"spec":{"replicas":3}}'
```

### Delete resources
```bash
kubectl delete pod nginx
kubectl delete deployment nginx
kubectl delete service nginx
```

### Force delete a Pod
```bash
kubectl delete pod nginx --grace-period=0 --force
```

> Use force deletion only when you understand the consequences.

## 11. Labels and Selectors

### Add a label
```bash
kubectl label pod nginx app=web
```

### Update an existing label
```bash
kubectl label pod nginx app=api --overwrite
```

### Query by selector
```bash
kubectl get pods -l app=web
```

### Show labels
```bash
kubectl get pods --show-labels
```

## 12. Fast Manifest Generation Cheatsheet

```bash
# Pod
kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml

# Deployment
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > deployment.yaml

# Namespace
kubectl create namespace dev --dry-run=client -o yaml > namespace.yaml

# ConfigMap
kubectl create configmap app-config --from-literal=ENV=prod --dry-run=client -o yaml > configmap.yaml

# Secret
kubectl create secret generic db-secret --from-literal=username=admin --dry-run=client -o yaml > secret.yaml

# Job
kubectl create job hello --image=busybox --dry-run=client -o yaml -- echo hello > job.yaml

# CronJob
kubectl create cronjob hello --image=busybox --schedule="*/5 * * * *" --dry-run=client -o yaml -- echo hello > cronjob.yaml

# Service
kubectl expose pod nginx --port=80 --name=nginx-service --dry-run=client -o yaml > service.yaml
```

## CKA Speed Tips

1. **Generate first, modify second.** Use `--dry-run=client -o yaml` to create a starting manifest quickly.
2. **Verify selectors.** Services depend on labels matching selectors.
3. **Use the namespace explicitly.** Prefer `kubectl -n <namespace>` when the task specifies a namespace.
4. **Troubleshoot before changing.** Start with `get`, `describe`, `logs`, and `exec`.
5. **Know when to switch to YAML.** Imperative commands are fast for object creation, but many CKA tasks are easier by editing a generated manifest.

## References

- [kubectl command reference](https://kubernetes.io/docs/reference/kubectl/)
- [kubectl cheat sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
- [CKA certification](https://training.linuxfoundation.org/certification/certified-kubernetes-administrator-cka/)

> This repository is a practical command reference. Kubernetes command availability can vary with client/server versions, so always validate commands against the cluster version you are using.
