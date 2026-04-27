# Hyperledger-Indy-K8s

### Steps to setup the network

- 1 Master and 2 Worker Node Setup.
- Using 4 Validator Nodes of Indy.

*_Follow Below Steps sequentiall:_*

1. Apply namespace `indy` in K8s cluster.
```bash
kubectl apply -f indy-namespace.yaml
```
2. Apply the PV and PVC for Local Persistence on worker nodes. **Create the directory `indyNetworkFiles` at /mnt on worker nodes before applying this.**
```bash
kubectl apply -f Local-Persistence-Storage/
```
3. Apply genesis updater RBAC based service account.
```bash
kubectl apply -f Security/
```
4. Apply the genesis job.
```bash
kubectl apply -f Jobs/
``` 
5. Apply the env configs for indy-nodes.
```bash
kubectl apply -f Configs/
```
6. Apply the services for indy-nodes.
```bash
kubectl apply -f Services/
```
7. Apply the indy nodes based on statefulsets.
```bash
# for node1 & node2 on worker 1
kubectl apply -f Indy-Nodes/statefulSets-worker1/

# for node2 & node3 on worker 2
kubectl apply -f Indy-Nodes/statefulSets-worker2/
```
8. Apply the network-explorer based on von-network.
```bash
kubectl apply -f Network-Explorer/
```
