**Kubelet**: The Kubernetes agent that runs on each node. It ensures containers are running in a Pod as expected.
# Creating a single Pod
```bash
kubectl run <pod-name> --image=<image-name>
```
# Inspecting resources
## get
```bash
kubectl get <resource>
```
- pods: Lists all pods in the current namespace.
- all: Retrieves commonly used resource information
- nodes: lists all cluster nodes
- events: 
- services:
- ingress: 
`-l lave=value` to filter with labels.
`--show-labels` To see labels attached to the resources 
### Getting a Specific Resource
> you can use `/` or a space to get a specific resource
```bash
kubectl get <resource-type>/<resource-name>
```
Example 
```bash
kubectl get pod/my-nginx
```
### Output formats
use the `-o <outputformat>`
common output formats
- json
- yaml
- wide
## Describe
Show details of a specific resource or group of resources.
```bash
kubectl describe  <resource-type>/<resource-name>
```
## Logs
Prints the logs for a container in a pod.
```bash
kubectl logs <resource-type>/<resource-name>
```
you can write the container name after it
Example
```bash
kubectl logs pods/my-apache-856f76d9f8-wfzwb httpd
```

> `--tail`: to make it only show the last x of lines
> `--follow x`:
> `-all-containers=true`: To get all the logs of the container in a pod
> 	`-l app=my-apache`: To do label search 

# Creating deployment
```bash
kubectl create deployment my-nginx --image=nginx
```
# Scaling replicasets
```bash
kubectl scale deployment <deployment-name> --replicas <number>
```
# Expose
```bash
kubectl expose <resource-type>/<resource-name>
```
- `--port <port-number>`: Specifies the port to expose.(on the service)
- `--name <name>`: Assigns a DNS-compliant name to the created service.
- `--type <type>`: Defines the type of service (e.g., `ClusterIP`, `NodePort`, `LoadBalancer`, `ExternalName`).
- `--target-port`: This is the **port on the pod** that the traffic is forwarded to.
## Port forwarding
```bash
kubectl port-forward <resource-type>/<resource-name> <local-port>:<resource-port>
```
# To get in a pod
```bash
kubectl run <pod-name> --rm -it --image <image-name> -- bash
```
# Resource generator
To preview the changes a CLI command would make to the YAML file without applying, append the following flags:
```bash
--dry-run=client -o yaml
```
> there is another type of dry run which reflects what will happen on the server using 
> `--dry-run=server`
```
# Apply
is used to **create or update Kubernetes resources**.
```bash
kubectl apply -f <file-or-directory-or-url>
```
# YAML
docs and examples  https://kubernetes.io/docs/reference/#api-reference
```yaml
apiVersion:
kind: 
metadata:
spec:
```
Use `---` to separate multiple Kubernetes objects in a single YAML file.
## To discover available resource kinds,api versions, run:
```bash
kubectl api-resources
```
use the following to know what can be put in spec
```bash
kubectl explain <resource-type> --recursive
```
> you can dig deeper using the `.`

for more info use 
```bash
kubectl explain <resource-type>.spec
```
for details about specific key 
```bash
kubectl explain <resource-type>.spec.<the-key>
```
## kubectl apply
>Used to **create or update** Kubernetes resources from a YAML.

```bash
kubectl apply -f <filename>
```
- `-l label=value` to only apply to objects with a certain label.
- `---dry-run=client/server -o yaml` to see what will change without applying
## kubectl diff
To see difference between the live state of a resource in the cluster and the configuration in a local file
```bash
kubectl diff -f file.yaml
```
## Labels
> **labels** are **key-value pairs** attached to objects (like Pods, Services, etc.) to help **organize, select, and group** them.

### To add labels to running pods
```bash
kubectl label pod myapp key=value
```

They can be added inside the metadata
Example
```yaml
metadata:
  name: nginx-deployment
  labels:
    servers: dmz
```

## Selectors
Are used to do something with a group of object that has a label in the yaml file.
Example:a NodePort that has to direct traffic to a group of nginx pods we add the following to the specs of the service
```yaml
selector:
    app: app-nginx
```
and the following to be added to the specs of the deployment(read docs you don't understand that well)
```yaml
template
  selector:
    matchLabels:
      app: app-nginx
```