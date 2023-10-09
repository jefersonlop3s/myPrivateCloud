# Code-Server

Mais informações em https://hub.docker.com/r/linuxserver/code-server 

Para este momento, estou utilizando a versão 4.17.1 do code-server.

*Observações:*
1. Para todos os efeitos estou usando o domínio fictício `home.org`.

## Teste local 
Use o Docker Engine/Deskto, Podman ou Containerd (nerdctl) para executar o code-server em um container local. Exemplo:

```
nerdctl run -d --name=code-server \
  -e PUID=1000 \
  -e PGID=1000 \
  -e PASSWORD=thisismypassword \
  -p 8443:8443 \
  -v /caminho/para/appdata/config:/config \
  --restart unless-stopped \
  linuxserver/code-server:4.17.1
``` 

Se tudo ocorreu bem, você terá o code-server executando em um conteiner em seu desktop, você poderá acessar através da url https://127.0.0.1:8443.

## Kubernetes Deploy

Utilize os arquivos na sequencia para um deploy eficaz.

_E sim, eu sei.. eu poderia ter utilizado um unico manifesto e ok, está tudo bem._

### 01-namespace.yaml
Define o namespace

### 02-secret.yaml

Arquivo contendo as secret para acesso a aplicação após a implantação.

Para que consiga gerar sua hashed password, utilize a documentação presente na FAQ em https://github.com/coder/code-server/blob/main/docs/FAQ.md#can-i-store-my-password-hashed.

Você precisará instalar o pacote `argon`, `npm` e o `npx` e basicamente executar os passos presentes no link indicado e colocar no valor de sua variável com nome "cs0-hashed" neste arquivo.

**Gerando a hashed**: 

```
echo -n "thisismypassword" | npx argon2-cli -e
$argon2i$v=19$m=4096,t=3,p=1$wst5qhbgk2lu1ih4dmuxvg$ls1alrvdiwtvzhwnzcm1dugg+5dto3dt1d5v9xtlws4
```

Caso queira utilizar apenas uma senha, você pode realizar a codificação dela utilizando o seguinte comando e obter a mesma com uma hash em base64:

```
echo "thisismypassword" | base64
dGhpc2lzbXlwYXNzd29yZAo=
```
Neste arquivo, estou deixando as variáveis `cs0-passwd` *(base64)* e `cs0-hashed` *(hashed)* preenchidas com estes valores, portando, para um uso real, substitua gerando com os valores que queira definir para a senha de acesso ao code-server após a implantação.

Estes exemplos foram tirados da documentação oficial presente na FAQ indicada.

### 03-certificate.yaml

Este arquivo é opcional e você pode gerar ou usar sua própria secret com seu certificado e sua chave ssl auto-assinada ou não para prover criptográfia para uso do protocolo https para acesso a sua aplicaçao.

Utilizo desta forma neste momento pois possuo o cert-manager implantado em meu cluster kubernetes do qual me provê certificados selfSigned para meu ingress.

### 04-pv.yaml
Você poderá também utilizar um PersistentVolume que faça mais sentido para você, porém, neste cluster local, estou utilizando armazenamento local. 

Modifique conforme sua necessidade.

### 05-pvc.yaml
Você poderá, também, utilizar ou não este manifesto adaptando a suas necessidades ou alterá-lo completamente, sinta-se a vontade se este não for para você.

### 06-deployment.yaml
Para que você obtenha sucesso, é necessário que você tenha o namespace, as secrets, um persistentVolume e um persistentVolumeClaim presente e apropriado para que este manifesto possa ser aplicado ao seus cluster.

Modifique-o conforme a sua necessidade.

Acabei por definir os recursos iniciais de cpu e memória que o pod poderá querer, como também defini um limite para este, também sinta-se a vontade para modificar conforme sua vontade e necessidade.

### 07-service.yaml
Por padrão, esta imagem já provê um ssl auto-assinado para o container/pod do qual após subí-lo, você terá a porta 8443 como padrão para acesso ao container.
Neste manifesto de serviço, defino a porta 80 padrão que através da target possa se comunicar com o Pod, e então, também, defino a 443 que será padrão para uso com o ingress. 

### 08-ingress.yaml
As anotações para meu cluster, definem o Traefik como ingressClass, caso não seja o que deseja ou use, modifque. Outras informações são relacionadas ao cert-manager, que, como dito anteriormente, gera o certificado para cada ingress que publico para minhas aplicações.

As chaves em tls, definem a secret que será usada, que se trata da mesma criada anteriormente em 03-certificate.yaml.

## À fazer.. Melhorias

1. Provavelmente gerar uma nova imagem, tendo como base a `linuxserver/code-server:4.17.1` e implementar os seguintes pacotes e binários:
* Java/OpenJDK
* Python
* NPM
* Algo mais que sinta falta

2. Descobrir uma forma de utilizar o Freeipa ou AD através de LDAP para autenticação de usuários.

3. Criacão de espaços de usuário, dinâmicamente, se o passo 2 for possível.

4. Melhorar a segurança para confinamento do Workspace e não permitir que o usuário da aplicação acesse um nível acima na hierarquia do sistema no container *(LFHS: Linux Filesystem Hierarchy Stantdard)*

5. Provável eu não faça nada disso _(Passo 1 eu farei)_, ou, ao menos tente os demais.