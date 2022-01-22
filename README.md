<div align="center">
<img src="https://user-images.githubusercontent.com/47891196/139104117-aa9c2943-37da-4534-a584-e4e5ff5bf69a.png" width="350px" />
</div>

# K3D-Estudos
Estudo da ferramenta k3d via k8s

# K3D

[linke](https://k3d.io/v5.2.2/#installation)

> **_Install_**

```
wget -q -O - https://raw.githubusercontent.com/rancher/k3d/main/install.sh | bash
```
```
curl -s https://raw.githubusercontent.com/rancher/k3d/main/install.sh | bash
```
* Create a cluster named mycluster with just a single server node:

```
k3d cluster create
```
* erro no docker rodar esse comando:

```
sudo chmod 666 /var/run/docker.sock
```
* Use the new cluster with kubectl, e.g.:

```
kubectl get nodes
```
* Ele cria um proxy para fazer o balancer
```
docker container ls
```
# Criar um cluster k8s sem proxy e sem balancer com um nome

```
k3d cluster create meucluster --no-lb
```
```
docker container ls
```
```
kubectl get nodes
NAME                      STATUS   ROLES                  AGE   VERSION
k3d-meucluster-server-0   Ready    control-plane,master   45s   v1.21.7+k3s1
```
# Listar os seus cluster

```
k3d cluster list
```

# Excluuir os cluster

* Excluir o cluster default

```
k3d cluster delete
```

* Listar

```
k3d cluster list
NAME         SERVERS   AGENTS   LOADBALANCER
meucluster   1/1       0/0      false
```
# Excluir o cluster especifico pelo nome

* Excluir pelo nome

```
k3d cluster delete meucluster
```

* Listar

```
k3d cluster list
NAME   SERVERS   AGENTS   LOADBALANCER
```

# Criar um cluster com alta disponibilidade

> **_Cenario de cluster local para teste e desenvolvimento_**

* Criar

```
k3d cluster create meucluster --servers 3 --agents 3
```
* Listar os nodes

```
kubectl get nodes
```
```
NAME                      STATUS   ROLES                       AGE   VERSION
k3d-meucluster-agent-0    Ready    <none>                      49s   v1.21.7+k3s1
k3d-meucluster-agent-1    Ready    <none>                      49s   v1.21.7+k3s1
k3d-meucluster-agent-2    Ready    <none>                      48s   v1.21.7+k3s1
k3d-meucluster-server-0   Ready    control-plane,etcd,master   79s   v1.21.7+k3s1
k3d-meucluster-server-1   Ready    control-plane,etcd,master   67s   v1.21.7+k3s1
k3d-meucluster-server-2   Ready    control-plane,etcd,master   54s   v1.21.7+k3s1
```
* Listar

```
docker container ls
CONTAINER ID   IMAGE                      COMMAND                  CREATED         STATUS              PORTS                             NAMES
3e05cc30cfa8   rancher/k3d-proxy:5.2.2    "/bin/sh -c nginx-pr…"   2 minutes ago   Up About a minute   80/tcp, 0.0.0.0:36257->6443/tcp   k3d-meucluster-serverlb
23b4fbf9bc4e   rancher/k3s:v1.21.7-k3s1   "/bin/k3s agent"         2 minutes ago   Up About a minute                                     k3d-meucluster-agent-2
90df9b3d6109   rancher/k3s:v1.21.7-k3s1   "/bin/k3s agent"         2 minutes ago   Up About a minute                                     k3d-meucluster-agent-1
b06761a3ed85   rancher/k3s:v1.21.7-k3s1   "/bin/k3s agent"         2 minutes ago   Up About a minute                                     k3d-meucluster-agent-0
8abc7a160fea   rancher/k3s:v1.21.7-k3s1   "/bin/k3s server --t…"   2 minutes ago   Up About a minute                                     k3d-meucluster-server-2
12c85947c338   rancher/k3s:v1.21.7-k3s1   "/bin/k3s server --t…"   2 minutes ago   Up 2 minutes                                          k3d-meucluster-server-1
0b6718b8f0d8   rancher/k3s:v1.21.7-k3s1   "/bin/k3s server --c…"   2 minutes ago   Up 2 minutes                                          k3d-meucluster-server-0
```
# Pod

* Criar o Pod

```
kubectl create -f pod.yaml
pod/meupod created
```

* Listar o Pod

```
kubectl get pods
NAME     READY   STATUS    RESTARTS   AGE
meupod   1/1     Running   0          70s

```
* Describe no pod especifico

```
kubectl describe pod/meupod
```
* Expondo minha porta local para meu pod, tudo que acessar da minha porta local 8080 sera redirecionada pra porta 80 do meu pod

```
kubectl port-forward pod/meupod 8080:80
```
```
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
```
* Teste no navegador

```
localhot:8080
```
```Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
Handling connection for 8080
Handling connection for 8080
```

# Deletar o Pod

* Deletar

```
kubectl delete pod meupod
pod "meupod" deleted
```
* Listar

```
kubectl get pods
```

# Label e Selectors

* Label: recursos de chave e valor que adiciono um objeto para marcar ele, marcando meus objetos
* Selector: usado pra selecionar objetos que possuem uma label, selecionando meus objetos

* kubectl apply usado para quando meu manifesto foi alterado

```
kubectl apply -f pod.yaml
pod/meupod created
pod/meupod-label created
```
* Simulando nova tentativa de alteração

```
kubectl apply -f pod.yaml
pod/meupod unchanged
pod/meupod-label unchanged
```
```
kubectl get pods
NAME           READY   STATUS    RESTARTS   AGE
meupod         1/1     Running   0          67s
meupod-label   1/1     Running   0          67s
```
* Pod sem Label

```
kubectl describe pod meupod
Name:         meupod
Namespace:    default
Priority:     0
Node:         k3d-meucluster-agent-1/172.20.0.5
Start Time:   Wed, 19 Jan 2022 12:31:56 -0300
Labels:       <none>
```

* Pod com Label
```
kubectl describe pod meupod-label
Name:         meupod-label
Namespace:    default
Priority:     0
Node:         k3d-meucluster-server-1/172.20.0.2
Start Time:   Wed, 19 Jan 2022 12:31:56 -0300
Labels:       app=web
```
* Buscar pela label o pod
```
kubectl get pods -l app=web
NAME           READY   STATUS    RESTARTS   AGE
meupod-label   1/1     Running   0          6m3s
```

# Controlador ReplicaSet

* replicas desejadas = replicas corrente
* Criar arquivo replicaset

* Listar

```
kubectl get pods
NAME           READY   STATUS    RESTARTS   AGE
meupod         1/1     Running   0          33m
meupod-label   1/1     Running   0          33m
```
* Deletar os pods

```
kubectl delete -f pod.yaml
pod "meupod" deleted
pod "meupod-label" deleted
```
* Validar se deletou

```
kubectl get pods
No resources found in default namespace.
```

* Criar o replicaset

```
kubectl apply -f replicaset.yaml 
replicaset.apps/meureplicaset created
```
* Listar se criou meu pod

```
kubectl get pods
NAME                  READY   STATUS    RESTARTS   AGE
meureplicaset-jbc2q   1/1     Running   0          24s
```
* Listar se criou meu replicaset

```
kubectl get replicaset
NAME            DESIRED   CURRENT   READY   AGE
meureplicaset   1         1         1       97s
```
* Teste deletando pod

```
kubectl delete pod meureplicaset-jbc2q
pod "meureplicaset-jbc2q" deleted
```
* Após deletar ele recriou um novo pod `-hash` diferente

```
kubectl get pods
NAME                  READY   STATUS    RESTARTS   AGE
meureplicaset-6b4q2   1/1     Running   0          12s
```
* Aplicar as replicas do meu arquivo alterado

```
kubectl apply -f replicaset.yaml 
replicaset.apps/meureplicaset configured
```
* Listar os Pods

```
kubectl get pods
NAME                  READY   STATUS    RESTARTS   AGE
meureplicaset-55b7l   1/1     Running   0          47s
meureplicaset-6b4q2   1/1     Running   0          5m40s
meureplicaset-8fzf4   1/1     Running   0          47s
meureplicaset-k2wzl   1/1     Running   0          47s
```

* Listar meu replicaset

```
kubectl get replicaset
NAME            DESIRED   CURRENT   READY   AGE
meureplicaset   4         4         4       9m52s
```
* Dessa forma garantimos escalabilidade e reseliencia

* Outra forma de alterar o scale de replicas

```
kubectl scale replicaset meureplicaset --replicas 10
replicaset.apps/meureplicaset scaled

```
* Listar meus pods

```
kubectl get pods
NAME                  READY   STATUS              RESTARTS   AGE
meureplicaset-55b7l   1/1     Running             0          4m22s
meureplicaset-5b4t4   1/1     Running             0          9s
meureplicaset-6b4q2   1/1     Running             0          9m15s
meureplicaset-8fzf4   1/1     Running             0          4m22s
meureplicaset-cljsx   1/1     Running             0          9s
meureplicaset-d9kzs   1/1     Running             0          9s
meureplicaset-k2wzl   1/1     Running             0          4m22s
meureplicaset-lc4lm   1/1     Running             0          9s
meureplicaset-lpl57   0/1     ContainerCreating   0          9s
meureplicaset-t8ntz   1/1     Running             0          9s
```
* Listar meus replicaset

```
kubectl get replicaset
NAME            DESIRED   CURRENT   READY   AGE
meureplicaset   10        10        10      13m
```
> **_Obs:_**
* O comando acima não altera seu arquivo replicaset.yaml então ideal sempre alterar por ele se a sua intenção for manter esse numero, caso seja pra teste pode optar por esse comando!

* Se alguém executar novamente kubeclt apply em seu arquivo ele volta ao numero que o arquivo esta setado

```
kubectl apply -f replicaset.yaml 
replicaset.apps/meureplicaset configured
```
* Terminate por conta do apply

```
kubectl get pods
NAME                  READY   STATUS        RESTARTS   AGE
meureplicaset-55b7l   1/1     Running       0          8m54s
meureplicaset-5b4t4   0/1     Terminating   0          4m41s
meureplicaset-6b4q2   1/1     Running       0          13m
meureplicaset-8fzf4   1/1     Running       0          8m54s
meureplicaset-cljsx   0/1     Terminating   0          4m41s
meureplicaset-k2wzl   1/1     Running       0          8m54s
meureplicaset-lc4lm   0/1     Terminating   0          4m41s
meureplicaset-lpl57   0/1     Terminating   0          4m41s
meureplicaset-t8ntz   0/1     Terminating   0          4m41s
```
* Listar e validar o apply

```
kubectl get pods
NAME                  READY   STATUS    RESTARTS   AGE
meureplicaset-55b7l   1/1     Running   0          9m45s
meureplicaset-6b4q2   1/1     Running   0          14m
meureplicaset-8fzf4   1/1     Running   0          9m45s
meureplicaset-k2wzl   1/1     Running   0          9m45s
```

* Alterandom a versão do meu pod pra green e aplicando o apply

```
kubectl apply -f replicaset.yaml 
replicaset.apps/meureplicaset configured
```

* Listar meus pods

```
kubectl get pods
NAME                  READY   STATUS    RESTARTS   AGE
meureplicaset-55b7l   1/1     Running   0          17m
meureplicaset-6b4q2   1/1     Running   0          22m
meureplicaset-8fzf4   1/1     Running   0          17m
meureplicaset-k2wzl   1/1     Running   0          17m
```
* Aplicando o port-forward em um pod 

```
kubectl port-forward pod/meureplicaset-55b7l 8080:80
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
```

* Test localhost

```
localhost:8080

PÁGINA DE TESTE
Servidor que processou a requisição - meureplicaset-55b7l
```
> **_Obs_**

* O replicaset ele não gerencia a troca de versão, para fazer a troca de versão usando o replicaset eu teria que altera os pods 1 a 1 e excluir os antigos! O ideal é usar o Deployment para esses casos que ele gerencia meu replicaset.

* Deletando om pod que foi alterado

```
kubectl delete pod meureplicaset-55b7l
pod "meureplicaset-55b7l" deleted
```
* Tete local e aplicando o port-forward, funcionou

```
 kubectl port-forward pod/meureplicaset-jzwm8 8080:80
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
Handling connection for 8080
Handling connection for 8080

PÁGINA DE TESTE
Servidor que processou a requisição - meureplicaset-jzwm8
```
# Deployment

> **_Deployment para gerenciar meu replicaset_**

* Deployment gerencia as versões do meu replicaset, v1, v2 e etc... e cria um novo replicaset ela não deleta!
* Quando eu trabhlo com deployment automaticamente ele vai criar meu replicaset.

* Aplicando apply no meu deployment

```
kubectl apply -f deployment.yaml 
deployment.apps/meureplicaset created
```
* Listar meus pods que agora vou ter tando do replicaset quanto do deployment

```
kubectl get pods
NAME                             READY   STATUS    RESTARTS   AGE
meudeployment-77c5f4d98c-bnxvt   1/1     Running   0          38s
meudeployment-77c5f4d98c-dcnt8   1/1     Running   0          38s
meudeployment-77c5f4d98c-f69nn   1/1     Running   0          38s
meudeployment-77c5f4d98c-k9srw   1/1     Running   0          38s
meureplicaset-6b4q2              1/1     Running   0          46m
meureplicaset-77c5f4d98c-c2r4n   1/1     Running   0          5m23s
meureplicaset-77c5f4d98c-pnw5b   1/1     Running   0          5m23s
meureplicaset-77c5f4d98c-qz6bf   1/1     Running   0          5m23s
meureplicaset-77c5f4d98c-wq8k6   1/1     Running   0          5m23s
meureplicaset-8fzf4              1/1     Running   0          41m
meureplicaset-jzwm8              1/1     Running   0          16m
meureplicaset-k2wzl              1/1     Running   0          41m
```
* Listar apenas os replicas set

```
kubectl get replicaset
NAME                       DESIRED   CURRENT   READY   AGE
meudeployment-77c5f4d98c   4         4         4       109s
meureplicaset              4         4         4       50m
meureplicaset-77c5f4d98c   4         4         4       6m34s
```

* Listar apenas os deployments

```
kubectl get deployments
NAME            READY   UP-TO-DATE   AVAILABLE   AGE
meudeployment   4/4     4            4           2m46s
meureplicaset   4/4     4            4           7m31s
```

* Deletando o replicaset

```
kubectl delete -f replicaset.yaml 
replicaset.apps "meureplicaset" deleted
```
* Listar o resultado do delete

```
kubectl get pods
NAME                             READY   STATUS    RESTARTS   AGE
meudeployment-77c5f4d98c-bnxvt   1/1     Running   0          4m16s
meudeployment-77c5f4d98c-dcnt8   1/1     Running   0          4m16s
meudeployment-77c5f4d98c-f69nn   1/1     Running   0          4m16s
meudeployment-77c5f4d98c-k9srw   1/1     Running   0          4m16s
meureplicaset-77c5f4d98c-c2r4n   1/1     Running   0          9m1s
meureplicaset-77c5f4d98c-pnw5b   1/1     Running   0          9m1s
meureplicaset-77c5f4d98c-qz6bf   1/1     Running   0          9m1s
meureplicaset-77c5f4d98c-wq8k6   1/1     Running   0          9m1s
```
* Deletando os pods

```
kubectl delete pod meudeployment-77c5f4d98c-bnxvt meudeployment-77c5f4d98c-dcnt8 meudeployment-77c5f4d98c-f69nn 
pod "meudeployment-77c5f4d98c-bnxvt" deleted
pod "meudeployment-77c5f4d98c-dcnt8" deleted
pod "meudeployment-77c5f4d98c-f69nn" deleted
```
* Novos pods criados

```
 kubectl get pods
NAME                             READY   STATUS    RESTARTS   AGE
meudeployment-77c5f4d98c-4ctxk   1/1     Running   0          17s
meudeployment-77c5f4d98c-5d4zd   1/1     Running   0          17s
meudeployment-77c5f4d98c-fsknm   1/1     Running   0          17s
meudeployment-77c5f4d98c-k9srw   1/1     Running   0          6m58s
meureplicaset-77c5f4d98c-c2r4n   1/1     Running   0          11m
meureplicaset-77c5f4d98c-pnw5b   1/1     Running   0          11m
meureplicaset-77c5f4d98c-qz6bf   1/1     Running   0          11m
meureplicaset-77c5f4d98c-wq8k6   1/1     Running   0          11m
```

* Alterando pra 10 pods

```
kubectl apply -f deployment.yaml 
deployment.apps/meudeployment configured

kubectl get pods
NAME                             READY   STATUS    RESTARTS   AGE
meudeployment-77c5f4d98c-2tfcv   1/1     Running   0          7s
meudeployment-77c5f4d98c-4ctxk   1/1     Running   0          4m31s
meudeployment-77c5f4d98c-5d4zd   1/1     Running   0          4m31s
meudeployment-77c5f4d98c-7mxj7   1/1     Running   0          7s
meudeployment-77c5f4d98c-9pkq2   1/1     Running   0          7s
meudeployment-77c5f4d98c-fsknm   1/1     Running   0          4m31s
meudeployment-77c5f4d98c-k9srw   1/1     Running   0          11m
meudeployment-77c5f4d98c-qdcj4   1/1     Running   0          7s
meudeployment-77c5f4d98c-snvzg   1/1     Running   0          7s
meudeployment-77c5f4d98c-txt2x   1/1     Running   0          7s
meureplicaset-77c5f4d98c-c2r4n   1/1     Running   0          15m
meureplicaset-77c5f4d98c-pnw5b   1/1     Running   0          15m
meureplicaset-77c5f4d98c-qz6bf   1/1     Running   0          15m
meureplicaset-77c5f4d98c-wq8k6   1/1     Running   0          15m

```
* Aplicando a alteração blue no arquivo de deployment, ele vai encerrar e criar outros


```
kubectl apply -f deployment.yaml 
deployment.apps/meudeployment configured

kubectl get pods
NAME                             READY   STATUS              RESTARTS   AGE
meudeployment-77c5f4d98c-5j8mc   0/1     ContainerCreating   0          2s
meudeployment-77c5f4d98c-9s8sl   0/1     ContainerCreating   0          1s
meudeployment-77c5f4d98c-ggskf   0/1     ContainerCreating   0          2s
meudeployment-77c5f4d98c-t2rcl   0/1     ContainerCreating   0          1s
meudeployment-77c5f4d98c-vwwqp   0/1     ContainerCreating   0          2s
meudeployment-dc9f67696-69d7z    1/1     Running             0          2m
meudeployment-dc9f67696-6zg9l    1/1     Running             0          117s
meudeployment-dc9f67696-9gzjv    1/1     Running             0          117s
meudeployment-dc9f67696-k4dfr    1/1     Terminating         0          118s
meudeployment-dc9f67696-lp5d8    1/1     Running             0          119s
meudeployment-dc9f67696-lqmgh    1/1     Running             0          2m1s
meudeployment-dc9f67696-nmj68    1/1     Running             0          2m
meudeployment-dc9f67696-rdxmf    1/1     Running             0          2m
meudeployment-dc9f67696-vd5lh    1/1     Running             0          114s
meudeployment-dc9f67696-wqgcc    1/1     Terminating         0          2m
meureplicaset-77c5f4d98c-c2r4n   1/1     Running             0          19m
meureplicaset-77c5f4d98c-pnw5b   1/1     Running             0          19m
meureplicaset-77c5f4d98c-qz6bf   1/1     Running             0          19m
meureplicaset-77c5f4d98c-wq8k6   1/1     Running             0          19m
```

* Verificado meus replicaset alterados e antigos

```
kubectl get replicaset
NAME                       DESIRED   CURRENT   READY   AGE
meudeployment-77c5f4d98c   10        10        10      15m
meudeployment-dc9f67696    0         0         0       2m39s
meureplicaset-77c5f4d98c   4         4         4       20m
```
* Alterando a imagem

```
kubectl set image deployment meudeployment web=kubedevio/web-page:blue
deployment.apps/meudeployment image updated
```
* Voltando pra versão anterior e alterar pra blue no arquivo e dar apply

```
kubectl apply -f deployment.yaml 
deployment.apps/meudeployment configured

kubectl get replicaset
NAME                       DESIRED   CURRENT   READY   AGE
meudeployment-77c5f4d98c   0         0         0       21m
meudeployment-dc9f67696    10        10        10      8m53s
meureplicaset-77c5f4d98c   4         4         4       26m
```
* Testando a versão Blue

```
kubectl port-forward pod/meudeployment-dc9f67696-2mjp8 8080:80
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
Handling connection for 8080
Handling connection for 8080

localhost:8080
PÁGINA DE TESTE
Servidor que processou a requisição - meudeployment-dc9f67696-2mjp8
```
* Testando a versão Green

```
kubectl apply -f deployment.yaml 
deployment.apps/meudeployment configured

kubectl get pods
NAME                             READY   STATUS    RESTARTS   AGE
meudeployment-77c5f4d98c-48r6v   1/1     Running   0          19s
meudeployment-77c5f4d98c-4bfhn   1/1     Running   0          21s
meudeployment-77c5f4d98c-84m8b   1/1     Running   0          23s
meudeployment-77c5f4d98c-bq7cl   1/1     Running   0          20s
meudeployment-77c5f4d98c-fhr2b   1/1     Running   0          24s
meudeployment-77c5f4d98c-jdrnc   1/1     Running   0          24s
meudeployment-77c5f4d98c-nsrsj   1/1     Running   0          23s
meudeployment-77c5f4d98c-sztd4   1/1     Running   0          19s
meudeployment-77c5f4d98c-th226   1/1     Running   0          24s
meudeployment-77c5f4d98c-tqm2g   1/1     Running   0          17s
meureplicaset-77c5f4d98c-c2r4n   1/1     Running   0          32m
meureplicaset-77c5f4d98c-pnw5b   1/1     Running   0          32m
meureplicaset-77c5f4d98c-qz6bf   1/1     Running   0          32m
meureplicaset-77c5f4d98c-wq8k6   1/1     Running   0          32m

kubectl port-forward pod/meudeployment-77c5f4d98c-48r6v 8080:80
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80

localhost:8080

PÁGINA DE TESTE
Servidor que processou a requisição - meudeployment-77c5f4d98c-48r6v

```
* Fazendo o rollout para a versão Blue

```
kubectl rollout undo deployment meudeployment
deployment.apps/meudeployment rolled back

kubectl get pods
NAME                             READY   STATUS        RESTARTS   AGE
meudeployment-77c5f4d98c-4bfhn   0/1     Terminating   0          5m18s
meudeployment-77c5f4d98c-bq7cl   0/1     Terminating   0          5m17s
meudeployment-77c5f4d98c-jdrnc   0/1     Terminating   0          5m21s
meudeployment-77c5f4d98c-nsrsj   0/1     Terminating   0          5m20s
meudeployment-77c5f4d98c-sztd4   0/1     Terminating   0          5m16s
meudeployment-77c5f4d98c-th226   0/1     Terminating   0          5m21s
meudeployment-dc9f67696-dstbh    1/1     Running       0          11s
meudeployment-dc9f67696-kgqjm    1/1     Running       0          12s
meudeployment-dc9f67696-ljhlh    1/1     Running       0          15s
meudeployment-dc9f67696-prxtj    1/1     Running       0          10s
meudeployment-dc9f67696-pxpxv    1/1     Running       0          15s
meudeployment-dc9f67696-rwzj5    1/1     Running       0          11s
meudeployment-dc9f67696-tjg57    1/1     Running       0          10s
meudeployment-dc9f67696-tx2xl    1/1     Running       0          15s
meudeployment-dc9f67696-x8566    1/1     Running       0          15s
meudeployment-dc9f67696-zsnm9    1/1     Running       0          15s
meureplicaset-77c5f4d98c-c2r4n   1/1     Running       0          37m
meureplicaset-77c5f4d98c-pnw5b   1/1     Running       0          37m
meureplicaset-77c5f4d98c-qz6bf   1/1     Running       0          37m
meureplicaset-77c5f4d98c-wq8k6   1/1     Running       0          37m

kubectl get replicaset
NAME                       DESIRED   CURRENT   READY   AGE
meudeployment-77c5f4d98c   0         0         0       32m
meudeployment-dc9f67696    10        10        10      19m
meureplicaset-77c5f4d98c   4         4         4       37m

PÁGINA DE TESTE
Servidor que processou a requisição - meudeployment-dc9f67696-dstbh
```

* Aplicando o rollout novamente agora pora pegar e versão Green

```
kubectl rollout undo deployment meudeployment
deployment.apps/meudeployment rolled back

kubectl get pods
NAME                             READY   STATUS    RESTARTS   AGE
meudeployment-77c5f4d98c-6bsc8   1/1     Running   0          40s
meudeployment-77c5f4d98c-8k675   1/1     Running   0          37s
meudeployment-77c5f4d98c-dddnj   1/1     Running   0          39s
meudeployment-77c5f4d98c-fxdgv   1/1     Running   0          38s
meudeployment-77c5f4d98c-gzftz   1/1     Running   0          40s
meudeployment-77c5f4d98c-j7sth   1/1     Running   0          40s
meudeployment-77c5f4d98c-kd4p9   1/1     Running   0          36s
meudeployment-77c5f4d98c-sczdz   1/1     Running   0          34s
meudeployment-77c5f4d98c-z2xtt   1/1     Running   0          38s
meudeployment-77c5f4d98c-znnjr   1/1     Running   0          39s
meureplicaset-77c5f4d98c-c2r4n   1/1     Running   0          39m
meureplicaset-77c5f4d98c-pnw5b   1/1     Running   0          39m
meureplicaset-77c5f4d98c-qz6bf   1/1     Running   0          39m
meureplicaset-77c5f4d98c-wq8k6   1/1     Running   0          39m]

kubectl port-forward pod/meudeployment-77c5f4d98c-6bsc8  8080:80
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80

localhost:8080

PÁGINA DE TESTE
Servidor que processou a requisição - meudeployment-77c5f4d98c-6bsc8

kubectl get replicaset
NAME                       DESIRED   CURRENT   READY   AGE
meudeployment-77c5f4d98c   10        10        10      36m
meudeployment-dc9f67696    0         0         0       23m
meureplicaset-77c5f4d98c   4         4         4       41m
```
* Essa é uma forma de gerecdniar suas versões

# Tipo de Services

 # ClusterIP
  * Utlizado internamente no cluster k8s, só interno.
 # NodePort
  * Ele é acessivel de maneira externa, ele utiliza uma porta padrão entre ranger 30000 a 32767, de qualquer ip de máquinas do meu k8s para acessar.
 # LoadBalancer
  * Sempre quando é criado eu crio junto um loadbalancer de ip publico que vai me dar um ip de acesso ao service.

* Alterando o arquivo deployment.yaml e aplicando o apply para criar o service
```
kubectl apply -f deployment.yaml 
deployment.apps/meudeployment unchanged
service/web created
``` 

* Listar o service

```
kubectl get service
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.43.0.1      <none>        443/TCP        5h16m
web          NodePort    10.43.72.220   <none>        80:30182/TCP   63s
```

```
docker container ls
CONTAINER ID   IMAGE                      COMMAND                  CREATED       STATUS       PORTS                             NAMES
3e05cc30cfa8   rancher/k3d-proxy:5.2.2    "/bin/sh -c nginx-pr…"   5 hours ago   Up 5 hours   80/tcp, 0.0.0.0:36257->6443/tcp   k3d-meucluster-serverlb
23b4fbf9bc4e   rancher/k3s:v1.21.7-k3s1   "/bin/k3s agent"         5 hours ago   Up 5 hours                                     k3d-meucluster-agent-2
90df9b3d6109   rancher/k3s:v1.21.7-k3s1   "/bin/k3s agent"         5 hours ago   Up 5 hours                                     k3d-meucluster-agent-1
b06761a3ed85   rancher/k3s:v1.21.7-k3s1   "/bin/k3s agent"         5 hours ago   Up 5 hours                                     k3d-meucluster-agent-0
8abc7a160fea   rancher/k3s:v1.21.7-k3s1   "/bin/k3s server --t…"   5 hours ago   Up 5 hours                                     k3d-meucluster-server-2
12c85947c338   rancher/k3s:v1.21.7-k3s1   "/bin/k3s server --t…"   5 hours ago   Up 5 hours                                     k3d-meucluster-server-1
0b6718b8f0d8   rancher/k3s:v1.21.7-k3s1   "/bin/k3s server --c…"   5 hours ago   Up 5 hours                                     k3d-meucluster-server-0
```
```
docker inspect 8abc7a160fea
[
    {
        "Id": "8abc7a160fea46a06cb5a9e7f1cf6293d39e668189f1e5b72ab234231360304c",
        "Created": "2022-01-19T13:47:24.39552257Z",
        "Path": "/bin/k3s",
        "Args": [
            "server",
            "--tls-san",
            "0.0.0.0"
        ],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 16651,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2022-01-19T13:47:50.295342104Z",
            "FinishedAt": "0001-01-01T00:00:00Z"
        },
        "Image": "sha256:4cbf38ec7da6c499ecc50cc6c54ef8c89d88f02a95da66eab5e404ade7067c61",
        "ResolvConfPath": "/var/lib/docker/containers/8abc7a160fea46a06cb5a9e7f1cf6293d39e668189f1e5b72ab234231360304c/resolv.conf",
        "HostnamePath": "/var/lib/docker/containers/8abc7a160fea46a06cb5a9e7f1cf6293d39e668189f1e5b72ab234231360304c/hostname",
        "HostsPath": "/var/lib/docker/containers/8abc7a160fea46a06cb5a9e7f1cf6293d39e668189f1e5b72ab234231360304c/hosts",
        "LogPath": "/var/lib/docker/containers/8abc7a160fea46a06cb5a9e7f1cf6293d39e668189f1e5b72ab234231360304c/8abc7a160fea46a06cb5a9e7f1cf6293d39e668189f1e5b72ab234231360304c-json.log",
        "Name": "/k3d-meucluster-server-2",
        "RestartCount": 0,
        "Driver": "overlay2",
        "Platform": "linux",
        "MountLabel": "",
        "ProcessLabel": "",
        "AppArmorProfile": "unconfined",
        "ExecIDs": null,
        "HostConfig": {
            "Binds": [
                "k3d-meucluster-images:/k3d/images"
            ],
            "ContainerIDFile": "",
            "LogConfig": {
                "Type": "json-file",
                "Config": {}
            },
            "NetworkMode": "default",
            "PortBindings": null,
            "RestartPolicy": {
                "Name": "unless-stopped",
                "MaximumRetryCount": 0
            },
            "AutoRemove": false,
            "VolumeDriver": "",
            "VolumesFrom": null,
            "CapAdd": null,
            "CapDrop": null,
            "CgroupnsMode": "host",
            "Dns": null,
            "DnsOptions": null,
            "DnsSearch": null,
            "ExtraHosts": null,
            "GroupAdd": null,
            "IpcMode": "private",
            "Cgroup": "",
            "Links": null,
            "OomScoreAdj": 0,
            "PidMode": "",
            "Privileged": true,
            "PublishAllPorts": false,
            "ReadonlyRootfs": false,
            "SecurityOpt": [
                "label=disable"
            ],
            "Tmpfs": {
                "/run": "",
                "/var/run": ""
            },
            "UTSMode": "",
            "UsernsMode": "",
            "ShmSize": 67108864,
            "Runtime": "runc",
            "ConsoleSize": [
                0,
                0
            ],
            "Isolation": "",
            "CpuShares": 0,
            "Memory": 0,
            "NanoCpus": 0,
            "CgroupParent": "",
            "BlkioWeight": 0,
            "BlkioWeightDevice": null,
            "BlkioDeviceReadBps": null,
            "BlkioDeviceWriteBps": null,
            "BlkioDeviceReadIOps": null,
            "BlkioDeviceWriteIOps": null,
            "CpuPeriod": 0,
            "CpuQuota": 0,
            "CpuRealtimePeriod": 0,
            "CpuRealtimeRuntime": 0,
            "CpusetCpus": "",
            "CpusetMems": "",
            "Devices": null,
            "DeviceCgroupRules": null,
            "DeviceRequests": null,
            "KernelMemory": 0,
            "KernelMemoryTCP": 0,
            "MemoryReservation": 0,
            "MemorySwap": 0,
            "MemorySwappiness": null,
            "OomKillDisable": false,
            "PidsLimit": null,
            "Ulimits": null,
            "CpuCount": 0,
            "CpuPercent": 0,
            "IOMaximumIOps": 0,
            "IOMaximumBandwidth": 0,
            "MaskedPaths": null,
            "ReadonlyPaths": null,
            "Init": true
        },
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/cb96c84fe350cc6b81990214026425bafd785d9afd5cef66792483213f6ff4aa-init/diff:/var/lib/docker/overlay2/3bb879e7d57aea85917f35d00d3879cc2660d2f53c1ac2a92b69f9551aef86d8/diff:/var/lib/docker/overlay2/493d9625eb4716a708f8df0a1fdcaf2d0a76c0526ba06e00e78ffe03f505c5b1/diff:/var/lib/docker/overlay2/9fac58dd56eb3d3d98dd03d80fe2aaf08f119935c351cc324f0899b370b4ea2a/diff",
                "MergedDir": "/var/lib/docker/overlay2/cb96c84fe350cc6b81990214026425bafd785d9afd5cef66792483213f6ff4aa/merged",
                "UpperDir": "/var/lib/docker/overlay2/cb96c84fe350cc6b81990214026425bafd785d9afd5cef66792483213f6ff4aa/diff",
                "WorkDir": "/var/lib/docker/overlay2/cb96c84fe350cc6b81990214026425bafd785d9afd5cef66792483213f6ff4aa/work"
            },
            "Name": "overlay2"
        },
        "Mounts": [
            {
                "Type": "volume",
                "Name": "k3d-meucluster-images",
                "Source": "/var/lib/docker/volumes/k3d-meucluster-images/_data",
                "Destination": "/k3d/images",
                "Driver": "local",
                "Mode": "z",
                "RW": true,
                "Propagation": ""
            },
            {
                "Type": "volume",
                "Name": "9f94fe4b411b62ce60102b5ebf3d4362610e47ade7ab3326a2dc8b48ce023c36",
                "Source": "/var/lib/docker/volumes/9f94fe4b411b62ce60102b5ebf3d4362610e47ade7ab3326a2dc8b48ce023c36/_data",
                "Destination": "/var/lib/cni",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            },
            {
                "Type": "volume",
                "Name": "c05f15cd156c0a973360fb755f3e30a8adbb22eab578dfac1ce6736a442a8653",
                "Source": "/var/lib/docker/volumes/c05f15cd156c0a973360fb755f3e30a8adbb22eab578dfac1ce6736a442a8653/_data",
                "Destination": "/var/lib/kubelet",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            },
            {
                "Type": "volume",
                "Name": "b1bfc6b8ac4255ef1ea0d98988dd90d5b8c7150d96d1756f8f75030431a80dc8",
                "Source": "/var/lib/docker/volumes/b1bfc6b8ac4255ef1ea0d98988dd90d5b8c7150d96d1756f8f75030431a80dc8/_data",
                "Destination": "/var/lib/rancher/k3s",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            },
            {
                "Type": "volume",
                "Name": "00fd93a20317c98c49c54f11d1772050147833de57c3403902b1146fac96a743",
                "Source": "/var/lib/docker/volumes/00fd93a20317c98c49c54f11d1772050147833de57c3403902b1146fac96a743/_data",
                "Destination": "/var/log",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            }
        ],
        "Config": {
            "Hostname": "k3d-meucluster-server-2",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "K3S_TOKEN=cYQllmrkwUcZvFgQthNM",
                "K3S_URL=https://k3d-meucluster-server-0:6443",
                "K3S_KUBECONFIG_OUTPUT=/output/kubeconfig.yaml",
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/bin/aux",
                "CRI_CONFIG_FILE=/var/lib/rancher/k3s/agent/etc/crictl.yaml"
            ],
            "Cmd": [
                "server",
                "--tls-san",
                "0.0.0.0"
            ],
            "Image": "docker.io/rancher/k3s:v1.21.7-k3s1",
            "Volumes": {
                "/var/lib/cni": {},
                "/var/lib/kubelet": {},
                "/var/lib/rancher/k3s": {},
                "/var/log": {}
            },
            "WorkingDir": "",
            "Entrypoint": [
                "/bin/k3s"
            ],
            "OnBuild": null,
            "Labels": {
                "app": "k3d",
                "k3d.cluster": "meucluster",
                "k3d.cluster.imageVolume": "k3d-meucluster-images",
                "k3d.cluster.network": "k3d-meucluster",
                "k3d.cluster.network.external": "false",
                "k3d.cluster.network.id": "0bdcc3d52e6d708fad2aed0cc698709b7d3ddc628c39b80faf9e32e9693fa11d",
                "k3d.cluster.network.iprange": "172.20.0.0/16",
                "k3d.cluster.token": "cYQllmrkwUcZvFgQthNM",
                "k3d.cluster.url": "https://k3d-meucluster-server-0:6443",
                "k3d.role": "server",
                "k3d.server.api.host": "0.0.0.0",
                "k3d.server.api.hostIP": "0.0.0.0",
                "k3d.server.api.port": "36257",
                "k3d.server.init": "false",
                "k3d.version": "v5.2.2",
                "org.opencontainers.image.created": "2021-11-29T16:43:32Z",
                "org.opencontainers.image.revision": "ac70570999c566ac3507d2cc17369bb0629c1cc0",
                "org.opencontainers.image.source": "https://github.com/k3s-io/k3s.git",
                "org.opencontainers.image.url": "https://github.com/k3s-io/k3s"
            }
        },
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "c156bd655486d2955afbf7ba34ec628ade65769b6623a4184985995d77561022",
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "Ports": {},
            "SandboxKey": "/var/run/docker/netns/c156bd655486",
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "",
            "Gateway": "",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "",
            "IPPrefixLen": 0,
            "IPv6Gateway": "",
            "MacAddress": "",
            "Networks": {
                "k3d-meucluster": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": [
                        "8abc7a160fea",
                        "k3d-meucluster-server-2"
                    ],
                    "NetworkID": "0bdcc3d52e6d708fad2aed0cc698709b7d3ddc628c39b80faf9e32e9693fa11d",
                    "EndpointID": "eb782b75068e42d7fee28ce465f41241673dab81c23a9278bdaa20431cbbd8ff",
                    "Gateway": "172.20.0.1",
                    "IPAddress": "172.20.0.4",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:14:00:04",
                    "DriverOpts": null
                }
            }
        }
    }
]
```
* Ip + a porta 80:30182/TCP:

```
 "IPAddress": "172.20.0.4",
``` 

* Acessar no navegador, com refresh no navegador ele vai mudar de pod

```
172.20.0.4:30182

PÁGINA DE TESTE
Servidor que processou a requisição - meureplicaset-77c5f4d98c-wq8k6

PÁGINA DE TESTE
Servidor que processou a requisição - meureplicaset-77c5f4d98c-pnw5b
```
* Fazer um port bind da minha porta local para o container k3s

```
k3d cluster delete meucluster
INFO[0000] Deleting cluster 'meucluster'                
INFO[0006] Deleting cluster network 'k3d-meucluster'    
INFO[0006] Deleting image volume 'k3d-meucluster-images' 
INFO[0006] Removing cluster details from default kubeconfig... 
INFO[0007] Removing standalone kubeconfig file (if there is one)... 
INFO[0007] Successfully deleted cluster meucluster!
```
* Recriar o cluster co bind de porta

```
k3d cluster create --agents 3 --servers 3 -p "8080:30000@loadbalancer"
```
* Listar meu cluster com k3d

```
k3d cluster list
NAME          SERVERS   AGENTS   LOADBALANCER
k3s-default   3/3       3/3      true
```
* Listar os container

```
docker container ls
CONTAINER ID   IMAGE                      COMMAND                  CREATED         STATUS              PORTS                                                                          NAMES
8595450690d5   rancher/k3d-proxy:5.2.2    "/bin/sh -c nginx-pr…"   2 minutes ago   Up About a minute   80/tcp, 0.0.0.0:41413->6443/tcp, 0.0.0.0:8080->30000/tcp, :::8080->30000/tcp   k3d-k3s-default-serverlb
4f30245f8d6c   rancher/k3s:v1.21.7-k3s1   "/bin/k3s agent"         2 minutes ago   Up About a minute                                                                                  k3d-k3s-default-agent-2
2d26710703c9   rancher/k3s:v1.21.7-k3s1   "/bin/k3s agent"         2 minutes ago   Up About a minute                                                                                  k3d-k3s-default-agent-1
dbd772e3e732   rancher/k3s:v1.21.7-k3s1   "/bin/k3s agent"         2 minutes ago   Up About a minute                                                                                  k3d-k3s-default-agent-0
24320d5a94ad   rancher/k3s:v1.21.7-k3s1   "/bin/k3s server --t…"   2 minutes ago   Up About a minute                                                                                  k3d-k3s-default-server-2
b7848b6d2167   rancher/k3s:v1.21.7-k3s1   "/bin/k3s server --t…"   2 minutes ago   Up About a minute                                                                                  k3d-k3s-default-server-1
da3aaf718c2c   rancher/k3s:v1.21.7-k3s1   "/bin/k3s server --c…"   2 minutes ago   Up About a minute
```
* Agora consigo ver o bind da porta no container

```
tcp, :::8080->30000
```
* Alterando arquivo deployment e aplicando apply inserindo nodePort: 30000

```
kubectl apply -f deployment.yaml
deployment.apps/meudeployment created
service/web created

 kubectl get pods
NAME                             READY   STATUS              RESTARTS   AGE
meudeployment-77c5f4d98c-2gx8c   0/1     ContainerCreating   0          24s
meudeployment-77c5f4d98c-5k28m   0/1     ContainerCreating   0          24s
meudeployment-77c5f4d98c-9h5rd   0/1     ContainerCreating   0          24s
meudeployment-77c5f4d98c-grfsx   0/1     ContainerCreating   0          24s
meudeployment-77c5f4d98c-h2j2p   0/1     ContainerCreating   0          24s
meudeployment-77c5f4d98c-lkwfd   0/1     ContainerCreating   0          24s
meudeployment-77c5f4d98c-ltcwq   0/1     ContainerCreating   0          24s
meudeployment-77c5f4d98c-m7nmb   0/1     ContainerCreating   0          24s
meudeployment-77c5f4d98c-pbmv5   0/1     ContainerCreating   0          24s
meudeployment-77c5f4d98c-qtz7p   0/1     ContainerCreating   0          24s

kubectl get all
NAME                                 READY   STATUS    RESTARTS   AGE
pod/meudeployment-77c5f4d98c-2gx8c   1/1     Running   0          53s
pod/meudeployment-77c5f4d98c-5k28m   1/1     Running   0          53s
pod/meudeployment-77c5f4d98c-9h5rd   1/1     Running   0          53s
pod/meudeployment-77c5f4d98c-grfsx   1/1     Running   0          53s
pod/meudeployment-77c5f4d98c-h2j2p   1/1     Running   0          53s
pod/meudeployment-77c5f4d98c-lkwfd   1/1     Running   0          53s
pod/meudeployment-77c5f4d98c-ltcwq   1/1     Running   0          53s
pod/meudeployment-77c5f4d98c-m7nmb   1/1     Running   0          53s
pod/meudeployment-77c5f4d98c-pbmv5   1/1     Running   0          53s
pod/meudeployment-77c5f4d98c-qtz7p   1/1     Running   0          53s

NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
service/kubernetes   ClusterIP   10.43.0.1      <none>        443/TCP        23m
service/web          NodePort    10.43.50.118   <none>        80:30000/TCP   53s

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/meudeployment   10/10   10           10          53s

NAME                                       DESIRED   CURRENT   READY   AGE
replicaset.apps/meudeployment-77c5f4d98c   10        10        10      53s
```
* Novo teste mas agora sem ip e pelo meu localhost

```
localhost:8080

PÁGINA DE TESTE
Servidor que processou a requisição - meudeployment-77c5f4d98c-5k28m
```

* Alterando replicas de 10 pra 30

```
kubectl get pods
NAME                             READY   STATUS              RESTARTS   AGE
meudeployment-77c5f4d98c-2gx8c   1/1     Running             0          3m42s
meudeployment-77c5f4d98c-4n5lk   0/1     ContainerCreating   0          2s
meudeployment-77c5f4d98c-5dp7r   0/1     ContainerCreating   0          2s
meudeployment-77c5f4d98c-5jdg5   0/1     ContainerCreating   0          2s
meudeployment-77c5f4d98c-5k28m   1/1     Running             0          3m42s
meudeployment-77c5f4d98c-6g9q9   0/1     ContainerCreating   0          2s
meudeployment-77c5f4d98c-7jxr4   0/1     ContainerCreating   0          2s
meudeployment-77c5f4d98c-8spwk   0/1     ContainerCreating   0          2s
meudeployment-77c5f4d98c-9czbg   0/1     ContainerCreating   0          2s
meudeployment-77c5f4d98c-9h5rd   1/1     Running             0          3m42s
meudeployment-77c5f4d98c-9hk8z   0/1     ContainerCreating   0          2s
meudeployment-77c5f4d98c-cw47v   0/1     ContainerCreating   0          2s
meudeployment-77c5f4d98c-dxrxx   0/1     ContainerCreating   0          2s
meudeployment-77c5f4d98c-grfsx   1/1     Running             0          3m42s
meudeployment-77c5f4d98c-h2j2p   1/1     Running             0          3m42s
meudeployment-77c5f4d98c-kg55r   0/1     ContainerCreating   0          2s
meudeployment-77c5f4d98c-lkwfd   1/1     Running             0          3m42s
meudeployment-77c5f4d98c-ltcwq   1/1     Running             0          3m42s
meudeployment-77c5f4d98c-m7nmb   1/1     Running             0          3m42s
meudeployment-77c5f4d98c-nm8f6   0/1     ContainerCreating   0          2s
meudeployment-77c5f4d98c-pbmv5   1/1     Running             0          3m42s
meudeployment-77c5f4d98c-qtz7p   1/1     Running             0          3m42s
meudeployment-77c5f4d98c-rwbgn   0/1     ContainerCreating   0          2s
meudeployment-77c5f4d98c-t4ck7   0/1     ContainerCreating   0          2s
meudeployment-77c5f4d98c-vcqxx   0/1     ContainerCreating   0          2s
meudeployment-77c5f4d98c-vt8gd   0/1     ContainerCreating   0          2s
meudeployment-77c5f4d98c-wp8mf   0/1     ContainerCreating   0          2s
meudeployment-77c5f4d98c-z9w45   0/1     ContainerCreating   0          2s
meudeployment-77c5f4d98c-zcqd7   0/1     ContainerCreating   0          2s
meudeployment-77c5f4d98c-zxjcv   0/1     ContainerCreating   0          2s
```
* Teste alterando de blue pra green ou vice e versa, refresh no navegador http://localhost:8080, teste ok!

```
kubectl apply -f deployment.yaml
deployment.apps/meudeployment configured
service/web unchanged
```
# Deletando meu cluster

* Comando delete cluster

```
k3d cluster delete
INFO[0000] Deleting cluster 'k3s-default'               
INFO[0006] Deleting cluster network 'k3d-k3s-default'   
INFO[0006] Deleting image volume 'k3d-k3s-default-images' 
INFO[0006] Removing cluster details from default kubeconfig... 
INFO[0006] Removing standalone kubeconfig file (if there is one)... 
INFO[0006] Successfully deleted cluster k3s-default!
```
