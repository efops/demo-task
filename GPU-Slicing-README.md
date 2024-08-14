## Install the NVIDIA GPU Operator:
```shell
helm repo add nvidia https://helm.ngc.nvidia.com/nvidia
helm repo update
helm install gpu-operator nvidia/gpu-operator --namespace gpu-operator --create-namespace
```
## Enable time-slicing:
After installing the GPU Operator, you need to enable time-slicing. This is done by creating a ConfigMap:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: gpu-operator-config
  namespace: gpu-operator
data:
  gpu-feature-discovery-config: |-
    version: v1
    flags:
      - --mig-strategy=single
      - --gpu-feature-discovery-enable-mig-mode-device-filtering=true
  device-plugin-config: |-
    version: v1
    sharing:
      timeSlicing:
        renameByDefault: true
        failRequestsGreaterThanOne: false
        resources:
        - name: nvidia.com/gpu
          replicas: 8
```
## Apply the ConfigMap:
```shell
kubectl apply -f gpu-operator-config.yaml
```
## Restart the GPU Operator pods:
```shell
kubectl rollout restart deployment -n gpu-operator
```
## GPU Slicing with Karpenter:
Karpenter doesn't directly manage GPU slicing, but it can provision nodes with GPUs that are then managed by the NVIDIA GPU Operator. Here's how to set it up:
### Update your Karpenter Provisioner to include GPU instances:
```yaml
apiVersion: karpenter.sh/v1alpha5
kind: Provisioner
metadata:
  name: default
spec:
  requirements:
    - key: karpenter.k8s.aws/instance-family
      operator: In
      values: ["g4dn", "p3", "p4d"] # GPU instance families
    - key: karpenter.k8s.aws/instance-size
      operator: NotIn
      values: ["nano", "micro", "small"]
  limits:
    resources:
      nvidia.com/gpu: 8 # Adjust based on your needs
  providerRef:
    name: default
  ttlSecondsAfterEmpty: 30
```
### Create a NodePool for GPU instances:
```yaml
apiVersion: karpenter.sh/v1alpha5
kind: NodePool
metadata:
  name: gpu
spec:
  template:
    spec:
      requirements:
        - key: node.kubernetes.io/instance-type
          operator: In
          values: ["g4dn.xlarge", "g4dn.2xlarge", "p3.2xlarge"] # Adjust based on your needs
      nodeClassRef:
        name: default
```
### Apply these configurations:
```shell
kubectl apply -f provisioner.yaml
kubectl apply -f nodepool.yaml
```
Now, when you create pods that request GPU resources, Karpenter will provision nodes with GPUs, and the NVIDIA GPU Operator will manage the GPU slicing on these nodes.
### Using GPU Slicing in Workloads:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: gpu-pod
spec:
  containers:
  - name: gpu-container
    image: nvidia/cuda:11.6.2-base-ubuntu20.04
    command: ["sleep"]
    args: ["infinity"]
    resources:
      limits:
        nvidia.com/gpu: "0.5" # Request half a GPU
```
