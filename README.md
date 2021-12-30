## Springboot on kubernetes
![image](https://user-images.githubusercontent.com/42948627/147616791-cf907af5-53e2-4b44-b7f4-8bc6fb9a2441.png)

## Objective
Run a simple api on kubernetes pod.

## Install Minikube;

Minikube is a simplified kubernetes, in other words a ONE Node cluster, 
where the master and worker processes are on the same machine.

```
sudo -i
apt install virtualbox virtualbox-ext-pack
wget https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
chmod +x minikube-linux-amd64
mv minikube-linux-amd64 /usr/local/bin/minikube
minikube version
minikube start
```
![image](https://user-images.githubusercontent.com/42948627/147616579-37eee47f-66b2-462f-abe4-dcb19e77d008.png)

## Install Kubenertes

To use kubernetes commands install kubectl, on linux the instalation packages doens't
have an apt default, however follow the step by step;

```
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
sudo touch /etc/apt/sources.list.d/kubernetes.list
echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" | tee -a /etc/apt/sources.list.d/kubernetes.list
apt update
apt install kubectl
```
Test if kubernetes is working 

```
kubectl get all
```

![image](https://user-images.githubusercontent.com/42948627/147674243-29e3a795-993c-4143-a83d-6ce66ed02842.png)


## Create Default Service, User and deployments 

```
kubectl create -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-beta6/aio/deploy/recommended.yaml
```

## Create authentication

Create .yaml to dashboard authentication;

```
apiVersion: v1
kind: ServiceAccount
metadata:
    name: loja-admin
    namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: loja-admin
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: loja-admin
  namespace: kube-system
```

## References

https://www.amazon.com.br/gp/product/B08ZWQ6YMB/ref=ppx_yo_dt_b_d_asin_title_o00?ie=UTF8&psc=1

https://dev.to/techworld_with_nana/what-is-minikube-and-kubectl-setup-a-minikube-cluster-for-kubernetes-beginners-5gj3

https://kubernetes.io/docs/tasks/tools/#install-kubectl-on-linux

https://askubuntu.com/questions/944920/how-do-i-delete-sources-list-d-and-add-install-it-again


https://stackoverflow.com/questions/47695369/kubernete-dashboard-is-not-deploying