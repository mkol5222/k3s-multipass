# K3S cluster with multipass and k3sup


```shell
ark get k3sup

ssh-keygen -f k3s

PUBLIC_SSH_KEY_PATH=./k3s.pub PRIVATE_SSH_KEY_PATH=./k3s ./mk.sh

# connect
export KUBECONFIG=`pwd`/kubeconfig
kubectl get node

# metallb

helm repo add metallb https://metallb.github.io/metallb
helm repo update
kubectl create namespace metallb
helm install metallb metallb/metallb --namespace metallb

kubectl -n metallb get pod --watch

# look at multipass VMs subnet
multipass ls
# modify range for MetalLB in config:
code metallb-config.yml
kubectl apply -f metallb-config.yml

kubectl create deploy web --image nginx --replicas 3 --port 80

kubectl expose deploy web --type LoadBalancer

kubectl get services --all-namespaces

LBIP=$(kubectl get svc web -o json | jq -r '.status.loadBalancer.ingress[0].ip')
curl http://$LBIP -vvv

# remove
for I in master node1 node2 ; do echo $I; multipass delete -p $I; done

```

Based on:
https://dev.to/tomowatt/creating-a-k3s-cluster-with-k3sup-multipass-h26
https://picluster.ricsanfre.com/docs/metallb/
