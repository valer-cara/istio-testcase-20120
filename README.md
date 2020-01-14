# Demo for the STALE/CDS issue

## Setup: create cluster, setup istio
```
# Demo cluster
minikube start --vm-driver=kvm2 --profile=istio

# Create operator
kubectl apply -f 1-operator.yaml

# Wait until operator settles, creates CRDs, etc..
kubectl apply -f 2-cluster.yaml

# All good now, make sure to
# wait until all pods in istio-system namespace are Running
# then try the demos
```

## Demo good: stuff keeps in SYNC

```
# Create SE, wait for it to sync
kubectl apply -f ./3-demo-good-part1.yaml

# Create DR
kubectl apply -f ./3-demo-good-part2.yaml

# All should be fine
istioctl ps
```

## Demo bad: stuff goes to stale

```
# Reset the first demo
kubectl delete -f ./3-demo-good-part2.yaml
kubectl delete -f ./3-demo-good-part1.yaml

# All should be fine still, 2xcheck
istioctl ps

# Apply the bad testcase
kubectl apply -f ./3-demo-bad.yaml

# At this point i always managed to reproduce: STALE state on CDS
istioctl ps

# Notice that even if you delete 3-demo-bad.yaml, STALE CDS is still stuck
kubectl delete -f ./3-demo-bad.yaml
```

## Cleanup
```
# Ensure demo-bad was deleted, then force restart sidecars

minikube --profile=istio ssh
sudo pkill -f svc.cluster.local

# Check, all should be SYNCED, good
istioctl ps
```
