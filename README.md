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
minikube config set cpus 4
minikube config set memory 15368
minikube start
```
![image](https://user-images.githubusercontent.com/42948627/147942099-b2a54757-c3d9-46a5-873f-e3100185eaa2.png)

## Install Kubernetes

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

![image](https://user-images.githubusercontent.com/42948627/147764725-158a00df-086f-426f-bf64-d917fed11da9.png)


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

## Start Kubernetes Dashboard

```
kubectl proxy
```

![image](https://user-images.githubusercontent.com/42948627/147764876-649ff947-504e-4d35-9ca1-6e043776d3ca.png)


## Access Kubernetes Dashboard

The path to access;

http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/#/login

![image](https://user-images.githubusercontent.com/42948627/147765685-a6f0ab20-4397-48fd-8579-fca1d44161ac.png)

To find token login press;

```
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep loja-admin | awk '{print $1}')
```

Choose loja-admin and place it on token and enter;
![image](https://user-images.githubusercontent.com/42948627/147776918-5fc70522-0b71-4433-9d3d-afe54fcc8945.png)


![image](https://user-images.githubusercontent.com/42948627/147765821-4b5597a1-ae13-44df-9bf2-712661fc26bf.png)


## Create Postgres Pod

kubectl create -f postgres-deployments.yaml

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres
  labels:
    app: postgres
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:latest
        ports:
        - containerPort: 5432
        env:
          - name: POSTGRES_USER
            value: postgres
          - name: POSTGRES_DB
            value: dev
          - name: POSTGRES_PASSWORD
            value: postgres
```

kubectl create -f postgres-service.yaml

```
apiVersion: v1
kind: Service
metadata:
  name: postgres
  labels:
    run: postgres
spec:
  ports:
    - port: 5432
      targetPort: 5432
      protocol: TCP
  selector:
    app: postgres
```

kubectl create -f config-map.yaml

```
kind: ConfigMap 
apiVersion: v1 
metadata:
  name: postgres-configmap 
data:
  database_url: jdbc:postgresql://postgres:5432/dev
  database_user: postgres
  database_password: postgres
```

Postgres Pod is running;

![image](https://user-images.githubusercontent.com/42948627/147789470-b1de4e97-d0dd-40cc-bc01-9f81c024e814.png)

To path postgres to OS access;

```
kubectl port-forward svc/postgres 5000:5432
```

![image](https://user-images.githubusercontent.com/42948627/147789941-7c99a49a-7a4c-47ea-9780-8fd2f919ace6.png)

## Create api images

On Terminal path springboot-on-kubernetes/product-api type;

```
mvn dockerfile:build
```

result;

```
[INFO] Image will be built as loja/product-api:latest
[INFO] 
[INFO] Step 1/5 : FROM openjdk:8-jdk-alpine
[INFO] 
[INFO] Pulling from library/openjdk
[INFO] Digest: sha256:94792824df2df33402f201713f932b58cb9de94a0cd524164a0f2283343547b3
[INFO] Status: Image is up to date for openjdk:8-jdk-alpine
[INFO]  ---> a3562aa0b991
[INFO] Step 2/5 : VOLUME /tmp
[INFO] 
[INFO]  ---> Using cache
[INFO]  ---> 9879ec2eea38
[INFO] Step 3/5 : ARG JAR_FILE=target/product-api-0.0.1.jar
[INFO] 
[INFO]  ---> Using cache
[INFO]  ---> b021b1862223
[INFO] Step 4/5 : COPY ${JAR_FILE} app.jar
[INFO] 
[INFO]  ---> 428a6a410383
[INFO] Step 5/5 : ENTRYPOINT ["java","-jar","/app.jar"]
[INFO] 
[INFO]  ---> Running in ef6363c8af13
[INFO] Removing intermediate container ef6363c8af13
[INFO]  ---> e0abbb66888a
[INFO] Successfully built e0abbb66888a
[INFO] Successfully tagged loja/product-api:latest
[INFO] 
[INFO] Detected build of image with id e0abbb66888a
[INFO] Building jar: /home/andre/projetos/springboot-on-kubernetes/product-api/target/product-api-0.0.1-docker-info.jar
[INFO] Successfully built loja/product-api:latest

```
On Terminal path springboot-on-kubernetes/shopping-api type;

```
mvn dockerfile:build
```

result;

```
[INFO] Image will be built as loja/shopping-api:latest
[INFO] 
[INFO] Step 1/5 : FROM openjdk:8-jdk-alpine
[INFO] 
[INFO] Pulling from library/openjdk
[INFO] Digest: sha256:94792824df2df33402f201713f932b58cb9de94a0cd524164a0f2283343547b3
[INFO] Status: Image is up to date for openjdk:8-jdk-alpine
[INFO]  ---> a3562aa0b991
[INFO] Step 2/5 : VOLUME /tmp
[INFO] 
[INFO]  ---> Using cache
[INFO]  ---> 9879ec2eea38
[INFO] Step 3/5 : ARG JAR_FILE=target/shopping-api-0.0.1.jar
[INFO] 
[INFO]  ---> Using cache
[INFO]  ---> d7d56be60d4d
[INFO] Step 4/5 : COPY ${JAR_FILE} app.jar
[INFO] 
[INFO]  ---> Using cache
[INFO]  ---> c5d41af3415e
[INFO] Step 5/5 : ENTRYPOINT ["java","-jar","/app.jar"]
[INFO] 
[INFO]  ---> Using cache
[INFO]  ---> 9ff74adb4e55
[INFO] Successfully built 9ff74adb4e55
[INFO] Successfully tagged loja/shopping-api:latest
[INFO] 
[INFO] Detected build of image with id 9ff74adb4e55
[INFO] Building jar: /home/andre/projetos/springboot-on-kubernetes/shopping-api/target/shopping-api-0.0.1-docker-info.jar
[INFO] Successfully built loja/shopping-api:latest

```

On Terminal path springboot-on-kubernetes/user-api type;

```
mvn dockerfile:build
```

result;

```
[INFO] Image will be built as loja/user-api:latest
[INFO] 
[INFO] Step 1/5 : FROM openjdk:8-jdk-alpine
[INFO] 
[INFO] Pulling from library/openjdk
[INFO] Digest: sha256:94792824df2df33402f201713f932b58cb9de94a0cd524164a0f2283343547b3
[INFO] Status: Image is up to date for openjdk:8-jdk-alpine
[INFO]  ---> a3562aa0b991
[INFO] Step 2/5 : VOLUME /tmp
[INFO] 
[INFO]  ---> Using cache
[INFO]  ---> 9879ec2eea38
[INFO] Step 3/5 : ARG JAR_FILE=target/user-api-0.0.1.jar
[INFO] 
[INFO]  ---> Using cache
[INFO]  ---> 8840f8475ad9
[INFO] Step 4/5 : COPY ${JAR_FILE} app.jar
[INFO] 
[INFO]  ---> 62a2f2beaa9f
[INFO] Step 5/5 : ENTRYPOINT ["java","-jar","/app.jar"]
[INFO] 
[INFO]  ---> Running in fcf2bdacc34c
[INFO] Removing intermediate container fcf2bdacc34c
[INFO]  ---> ae7d438519fa
[INFO] Successfully built ae7d438519fa
[INFO] Successfully tagged loja/user-api:latest
[INFO] 
[INFO] Detected build of image with id ae7d438519fa
[INFO] Building jar: /home/andre/projetos/springboot-on-kubernetes/user-api/target/user-api-0.0.1-docker-info.jar
[INFO] Successfully built loja/user-api:latest
```
List docker image;

```
docker image ls
```

![image](https://user-images.githubusercontent.com/42948627/147886155-59fa9287-55bd-4e63-b134-9532b7d569b7.png)

## After created images, create api POD

On Path user-api/deply, product-api/deply, shopping-api/deploy type;

```
kubectl create -f configmap.yaml
kubectl create -f deployment.yaml
```

![image](https://user-images.githubusercontent.com/42948627/147946525-30e79f8a-bffa-4b8e-806b-ba3d28375a06.png)

## Access POD from OS

```
kubectl port-forward svc/user-api 8080:8080
```

result;

![image](https://user-images.githubusercontent.com/42948627/147949857-d3ee768c-7beb-4884-ae52-a4b7412cc4b6.png)

A route for example http://localhost:8080/user/1

result;

![image](https://user-images.githubusercontent.com/42948627/147949547-2cf3f12a-4a9a-428f-bd43-58021b76c323.png)

## Run api images with docker compose

```
docker-compose up
```

```
product_1   |   .   ____          _            __ _ _
product_1   |  /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
product_1   | ( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
product_1   |  \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
product_1   |   '  |____| .__|_| |_|_| |_\__, | / / / /
product_1   |  =========|_|==============|___/=/_/_/_/
product_1   |  :: Spring Boot ::        (v2.3.0.RELEASE)
product_1   | 
postgres_1  | 2022-01-02 18:51:25.540 UTC [63] LOG:  database system was shut down at 2022-01-02 18:51:25 UTC
postgres_1  | 2022-01-02 18:51:25.548 UTC [1] LOG:  database system is ready to accept connections
product_1   | 2022-01-02 18:51:25.746  INFO 1 --- [           main] com.santana.java.back.end.App            : Starting App v0.0.1 on 0f0f3e567f7c with PID 1 (/app.jar started by root in /)
product_1   | 2022-01-02 18:51:25.753  INFO 1 --- [           main] com.santana.java.back.end.App            : No active profile set, falling back to default profiles: default
shopping_1  | 
shopping_1  |   .   ____          _            __ _ _
shopping_1  |  /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
shopping_1  | ( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
shopping_1  |  \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
shopping_1  |   '  |____| .__|_| |_|_| |_\__, | / / / /
shopping_1  |  =========|_|==============|___/=/_/_/_/
shopping_1  |  :: Spring Boot ::        (v2.3.0.RELEASE)
shopping_1  | 
user_1      | 
user_1      |   .   ____          _            __ _ _
user_1      |  /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
user_1      | ( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
user_1      |  \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
user_1      |   '  |____| .__|_| |_|_| |_\__, | / / / /
user_1      |  =========|_|==============|___/=/_/_/_/
user_1      |  :: Spring Boot ::        (v2.3.0.RELEASE)
```

## References

https://www.amazon.com.br/gp/product/B08ZWQ6YMB/ref=ppx_yo_dt_b_d_asin_title_o00?ie=UTF8&psc=1

https://dev.to/techworld_with_nana/what-is-minikube-and-kubectl-setup-a-minikube-cluster-for-kubernetes-beginners-5gj3

https://kubernetes.io/docs/tasks/tools/#install-kubectl-on-linux

https://askubuntu.com/questions/944920/how-do-i-delete-sources-list-d-and-add-install-it-again

https://stackoverflow.com/questions/47695369/kubernete-dashboard-is-not-deploying
