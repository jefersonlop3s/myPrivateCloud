# K3s Confs

Este documento cobre a instalação do MetalLB em um cluster K3s existente, do qual, o mesmo esteja com o seu serviço de Load Balancer padrão ativo.

Se você ainda não implantou seu cluster k3s, indico olhar a documentação oficial para uma instalação limpa sem a necessidade de que se realize estes passos, pois, todos eles podem ser evitados simplesmente implementando o k3s desabilitando tanto o traefik, como o servicelb na implantação:

1. `curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC=”server --disable traefik 
--disable servicelb” sh -`

Se você está implantando agora e seguiu com o comando demonstrado, desabilitando tanto o traefik como o servicelb, você pode pular o **Passo 1, desabilitando o servicelb** e ir diretamente para o **Passo 2, implantando o MetalLB**.
## MetalLB

### Passo 1, desabilitando o servicelb
Edite o arquivo `/etc/systemd/system/k3s.service` alterando a linha `ExecStart=/usr/local/bin/k3s server` acrescentando a opção `--disable servicelb`:

```
ExecStart=/usr/local/bin/k3s server --disable servicelb
``` 

1. Faça o reload dos daemons:
  * `systemctl daemon-reload `

2. Faça o restart do serviço do `k3s`:
  * `systemctl restart k3s.service`

### Passo 2, implantando o MetalLB
Siga os passos informados em https://metallb.universe.tf/installation/. Porém, pode-se utilizar a seguinte sequência de comandos.

2.1.  Criação do namespace:

`kubectl apply -f namespace`

2.2. Implantação do MetallLB:

`kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.11/config/manifests/metallb-native.yaml`


Se tudo ocorreu bem, deverá ver uma saída em seu terminal como esta:

```
namespace/metallb-system configured
customresourcedefinition.apiextensions.k8s.io/addresspools.metallb.io created
customresourcedefinition.apiextensions.k8s.io/bfdprofiles.metallb.io created
customresourcedefinition.apiextensions.k8s.io/bgpadvertisements.metallb.io created
customresourcedefinition.apiextensions.k8s.io/bgppeers.metallb.io created
customresourcedefinition.apiextensions.k8s.io/communities.metallb.io created
customresourcedefinition.apiextensions.k8s.io/ipaddresspools.metallb.io created
customresourcedefinition.apiextensions.k8s.io/l2advertisements.metallb.io created
serviceaccount/controller created
serviceaccount/speaker created
role.rbac.authorization.k8s.io/controller created
role.rbac.authorization.k8s.io/pod-lister created
clusterrole.rbac.authorization.k8s.io/metallb-system:controller created
clusterrole.rbac.authorization.k8s.io/metallb-system:speaker created
rolebinding.rbac.authorization.k8s.io/controller created
rolebinding.rbac.authorization.k8s.io/pod-lister created
clusterrolebinding.rbac.authorization.k8s.io/metallb-system:controller created
clusterrolebinding.rbac.authorization.k8s.io/metallb-system:speaker created
configmap/metallb-excludel2 created
secret/webhook-server-cert created
service/webhook-service created
deployment.apps/controller created
daemonset.apps/speaker created
validatingwebhookconfiguration.admissionregistration.k8s.io/metallb-webhook-configuration created
```

...ou, utilizar-se do yaml presente na pasta v0.13.11:

`kubectl apply -f v0.13.11\metallb-native.yaml`

2.3.. Aplique a configuração de range de IPs:

`kubectl apply -f v0.13.11\metallb-ipaddresspool.yaml`

Defina a faixa de IP que gostaria de deixar para que o MetalLB utilize para seus Serviços que terão além de um ClusterIP, também, Serviços do tipo `LoadBalancer`, atribuindo assim, um IP em sua LAN, isto, antes de executar o comando acima.

Em meu caso, estou definindo os IP's de `192.168.5.210` à `192.168.5.250`.