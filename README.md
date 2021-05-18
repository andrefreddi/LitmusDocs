---


## Litmus


__Install__


Aplique o manifesto do Operador LitmusChaos:
```java
kubectl apply -f https://litmuschaos.github.io/litmus/litmus-operator-v1.13.5.yaml
```
 O comando acima instala todos os CRDs, a configuração da conta de serviço necessária e o operador do caos.
___


#### Verifique sua instalação


__Verifique se o ChaosOperator está em execução__
```java
kubectl get pods -n litmus
```




| Saida esperada|
| ------ |
| chaos-operator-ce-554d6c8f9f-slc8k 1/1 Running 0 6m41s |
|


__Verifique se os CRDs caos estão instalados__
```java
kubectl get crds | grep chaos
```
| Saida esperada|
| - |
| chaosengines.litmuschaos.io 2019-10-02T08:45:25Z|
| chaosexperiments.litmuschaos.io 2019-10-02T08:45:26Z |
| chaosresults.litmuschaos.io 2019-10-02T08:45:26Z |
|
___
__Instalar experimentos do caos__


 - Os experimentos do caos contêm os detalhes reais do caos. Esses experimentos são instalados no seu cluster como CRs do Kubernetes. Os Experimentos do Caos são agrupados como Gráficos do Caos e são publicados no Chaos Hub .


````java
kubectl apply -f https://hub.litmuschaos.io/api/chaos/1.13.5?file=charts/generic/experiments.yaml -n nginx
````


____
__Configurar conta de serviço__
 - Uma conta de serviço deve ser criada para permitir que o chaosengine execute experimentos no namespace do seu aplicativo. Copie o seguinte em um rbac.yamlmanifesto e execute kubectl apply -f rbac.yamlpara criar uma dessas contas no namespace nginx.
````java
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: pod-delete-sa
  namespace: nginx
  labels:
    name: pod-delete-sa
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-delete-sa
  namespace: nginx
  labels:
    name: pod-delete-sa
rules:
- apiGroups: [""]
  resources: ["pods","events"]
  verbs: ["create","list","get","patch","update","delete","deletecollection"]
- apiGroups: [""]
  resources: ["pods/exec","pods/log","replicationcontrollers"]
  verbs: ["create","list","get"]
- apiGroups: ["batch"]
  resources: ["jobs"]
  verbs: ["create","list","get","delete","deletecollection"]
- apiGroups: ["apps"]
  resources: ["deployments","statefulsets","daemonsets","replicasets"]
  verbs: ["list","get"]
- apiGroups: ["apps.openshift.io"]
  resources: ["deploymentconfigs"]
  verbs: ["list","get"]
- apiGroups: ["argoproj.io"]
  resources: ["rollouts"]
  verbs: ["list","get"]
- apiGroups: ["litmuschaos.io"]
  resources: ["chaosengines","chaosexperiments","chaosresults"]
  verbs: ["create","list","get","patch","update"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pod-delete-sa
  namespace: nginx
  labels:
    name: pod-delete-sa
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: pod-delete-sa
subjects:
- kind: ServiceAccount
  name: pod-delete-sa
  namespace: nginx
````
___
__Faça anotações em seu aplicativo__


 - Seu aplicativo deve ser anotado com litmuschaos.io/chaos="true". Como medida de segurança e também como meio de reduzir o raio de explosão, o ChaosOperator verifica essa anotação antes de invocar experimentos de caos no aplicativo. Substitua nginxpelo nome de sua implantação.


__Rodar o caos__


 - Aplique o manifesto ChaosEngine para acionar o experimento.
````java
kubectl apply -f chaosengine.yaml
````
__Observe os resultados do Caos__


 - Descreva o ChaosResult CR para saber o status de cada experimento. O status.verdicté definido para Awaitedquando o experimento está em andamento, eventualmente mudando para Passou Fail.


````java
kubectl describe chaosresult nginx-chaos-pod-delete -n nginx
````

## Resultados

__O que entendemos ao ultilizar a ferramenta?__

- O Litmus tem como grande semelhança ao uso da ferramenta "Chaos Monkey" tendo ferramentas de Chaos semelhantes porem com alguns diferencias em destaque.

__O que faz?__

- Litmus possui algumas ferramentas de uso de Chaos como a :

````
Pod Network Latency
Pod Network Loss
Pod Network Corruption
Container Kill
````
