# springboot-on-kubernetes
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

## References

https://www.amazon.com.br/gp/product/B08ZWQ6YMB/ref=ppx_yo_dt_b_d_asin_title_o00?ie=UTF8&psc=1

https://dev.to/techworld_with_nana/what-is-minikube-and-kubectl-setup-a-minikube-cluster-for-kubernetes-beginners-5gj3
