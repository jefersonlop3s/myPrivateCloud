# Notas

## NGINX IngressController

Implante o Nginx Ingress Controller apenas após a implantação do MetalLB

### Fix para IngressController

`sed -i 's/kind: Deployment/kind: DaemonSet/' controller-v1.9.3/nginx-deploy.yaml`

`sed -i 's/type: NodePort/type: LoadBalancer/' controller-v1.9.3/nginx-deploy.yaml`


### Fix para o DaemonSet
`DaemonSets` não possuem a especificação `strategy`, logo, precisamos comentar as seguintes linhas:

```
#strategy:
#    rollingUpdate:
#      maxUnavailable: 1
#    type: RollingUpdate
```

No arquivo `controller-v1.9.3/nginx-deploy.yaml` comente entre as linhas 410 e 413.

Este patch ou fix aplicamos, pois, a principal diferença entre um Deployment e um DaemonSet é o fato de o primeiro poder ser escalado horizontalmente, garantindo um número desejado de réplicas em execução no cluster, e o segundo, a garantia de uma cópia especifica sendo executada em cada  nó de nosso cluster, tendo apenas uma réplica por nó.

### Fix para ValidationWebhookConfiguration

Nas linha `659` alteramos o valor da chave `failurePolicy` de `Fail` para `Ignore`. 

Isto poderia ser feito também após a implantação se preferir, executando o seguinte comando, porém, seria necessário a instalação do `yq` antes:

```
kubectl get Validatingwebhookconfigurations ingress-nginx-admission -o=yaml | yq '.webhooks[].failurePolicy = "Ignore"' | kubectl apply -f -
```

Se precisar... siga as instruções para instalação do `yq`:
```
sudo wget -qO /usr/local/bin/yq https://github.com/mikefarah/yq/releases/latest/download/yq_linux_amd64

sudo chmod a+x /usr/local/bin/yq

yq --version
```

## Implantação após correções
Implante com:
```
kubectl apply -f controller-v1.9.3/nginx-deploy.yaml
```

Verifique os recursos no namespace `ingress-nginx`, onde deveremos ter algo como:

```
kubectl get all -n ingress-nginx
Warning: kubevirt.io/v1 VirtualMachineInstancePresets is now deprecated and will be removed in v2.
NAME                                       READY   STATUS      RESTARTS   AGE
pod/ingress-nginx-admission-create-nnhrs   0/1     Completed   0          42m
pod/ingress-nginx-admission-patch-9vxj4    0/1     Completed   1          42m
pod/ingress-nginx-controller-cbc66         1/1     Running     0          42m

NAME                                         TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)                      AGE
service/ingress-nginx-controller             LoadBalancer   10.43.140.162   192.168.5.211   80:32708/TCP,443:32513/TCP   42m
service/ingress-nginx-controller-admission   ClusterIP      10.43.202.86    <none>          443/TCP                      42m

NAME                                      DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
daemonset.apps/ingress-nginx-controller   1         1         1       1            1           kubernetes.io/os=linux   42m

NAME                                       COMPLETIONS   DURATION   AGE
job.batch/ingress-nginx-admission-create   1/1           3s         42m
job.batch/ingress-nginx-admission-patch    1/1           4s         42m
```
