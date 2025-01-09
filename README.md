# CompassUOL_DevSecOps_Atividades
- Repositório designado às atividades passadas pelos instrutores da CompassUOL na semana do dia 06 ao 10 do mês de Janeiro de 2025.

> [!Warning]
> ## O repositório apenas demonstra como executar algumas atividades de kubernetes. Sem o intuito de ensinar à preparar ou instalar o ambiente necessário.

# Visão Geral

> [!Tip]
> #### Site que monta um ambiente pronto para se familiarizar com Kubernetes: https://killercoda.com/kubernetes

> [!Tip]
> #### Material de apoio:
> #### https://kubernetes.io/releases/download/ <br>
> #### https://minikube.sigs.k8s.io/docs/start/?arch=%2Fwindows%2Fx86-64%2Fstable%2F.exe+download

1) - Criar um pod chamado "my-pod" usando uma imagem simples como "nginx" e verifique seu estado com os comandos de monitoramento do Kubernetes. [1)my-pod](#Pod))
2) - Implantar um Deployment chamado "my-deployment" com três réplicas de uma aplicação baseada na imagem "httpd". Atualize a imagem do Deployment para uma versão mais recente. [2)my-deployment](#Deployment))
3) - Criar um ConfigMap chamado "app-config" com uma variável de configuração personalizada. - Monte o ConfigMap em um pod e verifique se o valor foi aplicado corretamente. [3)ConfigMap](#ConfigMap))
4) - Criar um Secret chamado "app-secret" contendo informações sensíveis. Injete o Secret como uma variável de ambiente em um pod e teste se está acessível. [4)Secret](#Secret))
5) - Configurar um PersistentVolume de 1Gi de armazenamento local e vincule-o a um PersistentVolumeClaim. Monte o volume em um pod e salve arquivos para verificar a persistência. [5)PV, PVC](#PersistentVolume))
6) - Criar um serviço do tipo ClusterIP para um Deployment chamado "backend" e teste a conectividade interna entre pods usando o nome do serviço. [6)ClusterIP](#ClusterIP))
7) - Implantar um Job chamado "batch-job" que execute um comando simples e termine. Verifique os logs do Job para confirmar sua execução. [7)BatchJob](#BatchJob))
8) - Criar um Horizontal Pod Autoscaler para um Deployment chamado "hpa-deployment" e configure-o para escalar com base no uso de CPU. Aumente a carga e observe o escalonamento. [8)HPA](#HPA))
9) - Criar um serviço do tipo NodePort para expor externamente um Deployment chamado "webapp". Acesse o serviço usando o endereço IP do Minikube e a porta atribuída. [9)NodePort](#NodePort))
10) - Criar um pod chamado "restart-pod" com a política de reinício configurada como "OnFailure". - Provoque uma falha no pod e observe seu comportamento. [10)RestartPod](#RestartPod))

> [!Important]
> # Executando tarefas
> - Feito pelo terminal do Windows10

## 1) - Pod, no terminal execute os comandos abaixo.
```
kubectl run my-pod --image=nginx
kubectl get pods
kubectl describe pod my-pod
kubectl logs my-pod
```

- Após o primeiro comando, verifique como o exemplo abaixo:
![1-tarefa-mypod](https://github.com/user-attachments/assets/bee9a4e6-5da1-4a3f-bfc4-6eb9d760f03b)

## 2) - Deployment, no terminal execute os comandos abaixo.
```
kubectl create deployment my-deployment --image=httpd --replicas=3
kubectl get deployments
kubectl describe deployment my-deployment
kubectl set image deployment/my-deployment httpd=httpd:2.4  # Atualizar imagem
kubectl rollout status deployment/my-deployment  # Verificar progresso
```

- Exemplo após criar:
![2-tarefa-mydeployment](https://github.com/user-attachments/assets/dbe943de-43a1-4a41-9bbc-cade10343fc9)

## 3) - ConfigMap, no terminal execute os comandos abaixo.
```
kubectl create configmap app-config --from-literal=customKey=customValue
kubectl get configmaps
kubectl describe configmap app-config
```
> [!Important]
> Crie um arquivo chamado config-pod.yaml e insira o conteúdo abaixo neste arquivo:
```yaml
# config-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: config-pod
spec:
  containers:
  - name: nginx
    image: nginx
    env:
    - name: CONFIG_KEY
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: customKey
```

- No terminal execute:
```
kubectl apply -f config-pod.yaml
kubectl exec -it config-pod -- printenv CONFIG_KEY
```
- Exemplo:
![3-tarefa-ConfigMap](https://github.com/user-attachments/assets/6f285a1d-b9e4-492a-b408-f310b0d9604e)

## 4) - Secret, no terminal execute os comandos abaixo.
```
kubectl create secret generic app-secret --from-literal=username=admin --from-literal=password=pass123
kubectl get secrets
kubectl describe secret app-secret
```

> [!Important]
> Crie um arquivo chamado secret-pod.yaml e insira o conteúdo abaixo neste arquivo:
```yaml
# secret-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-pod
spec:
  containers:
  - name: nginx
    image: nginx
    env:
    - name: SECRET_USERNAME
      valueFrom:
        secretKeyRef:
          name: app-secret
          key: username
    - name: SECRET_PASSWORD
      valueFrom:
        secretKeyRef:
          name: app-secret
          key: password
```

- No terminal execute:
```
kubectl apply -f secret-pod.yaml
kubectl exec -it secret-pod -- printenv SECRET_USERNAME SECRET_PASSWORD
```

- Exemplo:
![4-tarefa-Secret](https://github.com/user-attachments/assets/b4aecd5e-b457-49f0-9e6b-c9f9099cb692)

## 5) - PersistentVolume e PersistentVolumeClaim.
> [!Important]
> Crie um arquivo chamado pv.yaml e insira o conteúdo abaixo neste arquivo:
```yaml
# pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: local-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
  - ReadWriteOnce
  hostPath:
    path: /data
---
# pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: local-pvc
spec:
  resources:
    requests:
      storage: 1Gi
  accessModes:
  - ReadWriteOnce
```

- No terminal, execute:
```
kubectl apply -f pv.yaml
kubectl apply -f pvc.yaml
kubectl get pv,pvc
```

> [!Important]
> Crie um arquivo chamado pod-with-pvc.yaml e insira o conteúdo abaixo neste arquivo:
```yaml
# pod-with-pvc.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pv-pod
spec:
  containers:
  - name: busybox
    image: busybox
    command: [ "sh", "-c", "echo Hello > /mnt/data/hello.txt && sleep 3600" ]
    volumeMounts:
    - mountPath: /mnt/data
      name: data-volume
  volumes:
  - name: data-volume
    persistentVolumeClaim:
      claimName: local-pvc
```

- No terminal, execute:
```
kubectl apply -f pod-with-pvc.yaml
kubectl exec -it pv-pod -- cat /mnt/data/hello.txt
```

Exemplo:
![5-tarefa-PV-PVC](https://github.com/user-attachments/assets/6d627d2d-34bf-44d7-9427-2acda39c9ff1)

## 6) - ClusterIP, no terminal execute os comandos abaixo.
```
kubectl expose deployment backend --port=80 --target-port=80 --type=ClusterIP
kubectl get services
kubectl exec -it <backend-pod-name> -- curl http://backend
```

Exemplo:
![6-tarefa-ClusterIP](https://github.com/user-attachments/assets/b285a668-d517-475d-94b0-5c62cba9adca)
![6-tarefaClusterIP2](https://github.com/user-attachments/assets/d3fec06a-f70b-49c4-a963-69fdf22b5daa)

## 7) - BatchJob.
> [!Important]
> Crie um arquivo chamado batch-job.yaml e insira o conteúdo abaixo neste arquivo:
```yaml
# batch-job.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: batch-job
spec:
  template:
    spec:
      containers:
      - name: busybox
        image: busybox
        command: ["echo", "Hello Kubernetes"]
      restartPolicy: Never
  backoffLimit: 4
```

- No terminal, execute:
```
kubectl apply -f batch-job.yaml
kubectl logs job/batch-job
```

Exemplo:
![7-tarefa-BatchJob](https://github.com/user-attachments/assets/16655d8e-7599-43fe-a83d-525c8b50a879)

## 8) - HPA (Horizontal Pod Autoscaler), no terminal execute os comandos abaixo.
```
kubectl autoscale deployment hpa-deployment --cpu-percent=50 --min=1 --max=5
kubectl describe hpa
# Simular carga
kubectl run load-generator --image=busybox -- sh -c "while true; do wget -q -O- http://hpa-deployment; done"
```

- Exemplo:
![8-tarefa-hpa](https://github.com/user-attachments/assets/aa8facab-ba9c-4884-8fb0-081e95579cdb)

## 9) - NodePort, no terminal execute os comandos abaixo.
```
kubectl expose deployment webapp --type=NodePort --port=80
kubectl get services
# Acessar via: Minikube IP + NodePort
minikube service webapp <Minikube IP>:<NodePort>
```

Exemplo:
![9-tarefa-NodePort](https://github.com/user-attachments/assets/e6d418bd-96d4-45f8-bc2d-bdcbe9a377cf)

Automaticamente deve abrir uma tela como esta:
![9-tarefa-NodePortWeb](https://github.com/user-attachments/assets/6f2ea224-c12e-4c91-92fe-4ea4899db36c)

## 10) - RestartPod.
> [!Important]
> Crie um arquivo chamado # restart-pod.yaml e insira o conteúdo abaixo neste arquivo:
```yaml
# restart-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: restart-pod
spec:
  restartPolicy: OnFailure
  containers:
  - name: busybox
    image: busybox
    command: ["false"]
```

- No terminal, execute:
```
kubectl apply -f restart-pod.yaml
kubectl describe pod restart-pod
```

- Exemplo:
![10-tarefa-RestartPod](https://github.com/user-attachments/assets/f255b53b-88ac-44ab-909b-7e9d6fe719a7)

## Parabéns! :tada:

Você Executou todas as tarefas. :partying_face:
