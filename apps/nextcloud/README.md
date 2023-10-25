# Nextcloud simple deploy
A implantaçao do Nextcloud aqui apresentada não se baseia no Nextcloud All in One, porém, mesmo assim, em uma imagem oficial disponibilizada por eles, onde você tem por padrão a instância do Nextcloud, podendo instalar os demais aplicativos disponibilizados pelo projeto via menu da administração em aplicativos.

Aqui, você também terá que gerar seu certificados via openssl para uso com o NGINX que está definido como um proxy reverso, estes certificados deverão ser disponibilizados ou colocados na pasta ./certs para ser consumido pelo serviço 'nginx' quando utilizar o compose.yaml via `docker-compose up -d` por exemplo.

Antes de mais nada, você precisará de um banco de dados postgres pronto e funcional com o usuário e banco de dados para o Nextcloud definidos e prontos para uso/consumo.

## Arquivos
* secret.txt
  * Contém dados sensíveis para o banco de dados Postgres e de administração de sua instância Nextcloud. Os dados aqui presentes são fictícios, logo, alterar os mesmos para algo real, faz bem.. caso prefira não utilizar o arquivos secret.txt, tudo bem, defina diretamente as informações no `compose.yaml` se lhes fizer sentido no momento de teste do Nextcloud.

* compose.yaml
  * Contém o manifesto para implantação do Nextcloud e do Nginx como Proxy Reverso.

* certs\
  * Pasta onde deverá ser colocado seu certificado para o servidor e sua chave para que você obtenha uma conexão segura entre seu navegador e o ambiente do Nextcloud. Os arquivos devem ser `server.crt` e `server.key`. Para mais informações de como gerar os mesmos, observe o conteúdo do arquivo `certs\README.md`

* nginx\nginx.conf
  * Arquivo de configuração do Nginx como proxy reverso. Fique a vontade para mudar principalmente as linhas #26 e #33 que definem qual a URL de acesso e/ou FQDN do host que será implantado o Nextcloud. Lembre de observar, se você possui um DNS local, cadastrar uma entrada A para que o IP do host onde está sendo executado este compose, possa responder corretamente ao que você definir como url de acesso ao Nextcloud.

* compose.yaml
  * Enfim, o arquivo de implantação de seu ambiente Nextcloud e do proxy reverso Nginx. Está definido IP fixos dos containers dentro de uma Network que é criada exclusivamente para este ambiente, Caso necessite de alterações, lembre-se de alterar a CIDR desta rede, IPs dos containers/services e atualizar as linhas #21 e #44 no arquivo `nginx\nginx.conf` para o proxy reverso. Não se preocupe, pois, provavelmente não precisará alterar nada por aqui, mas, sinta-se livre para isso =) 

E enfim, execute `docker-compose up -d` na raís deste diretório/pasta após cloná-lo para seu ambiente.


Inicie a implantação utilizando-se do endereço IP de seu host.

## Evitando possíveis problemas:
### Permissões de acesso Apache
Antes de iniciar a implantação, dê permissões totais ao diretório de implantação, não se preocupe, pois, a própria implantação regulará as permissões durante e após o processo:
`sudo chmod 777 -R /opt/nextcloud/{html,data}/ `

Sempre olhe os logs se o container não subir. 

`docker logs -f nextcloud-app-1`

Se houver algum inconveniente, após implantar, utilize `docker-compose down` para derrubar o ambiente, apagando os containers e siga o procedimento abaixo para caso de permissão negada, porém o procedimento acima deverá ter resolvido sem delongas e isso não acontecerá.. acredito... entao...

... Se ocorrer a excessão de Permission denied:
`/entrypoint.sh: 137: cannot create /var/www/html/nextcloud-init-sync.lock: Permission denied`

... prossiga com a remoção de todo o conteúdo desta primeira implantação com `sudo rm -rf /opt/nextcloud/html/* && sudo rm -rf /opt/nextcloud/data/*`.

Após isso, para evitar o problema, é necessário alterar as permissões do diretório que contém os volumes  para 'html' e/ou 'data' com:
`sudo chmod 777 -R /opt/nextcloud/{html,data}/ `

Isso vai permitir que o usuário 'www-data' do serviço apache executando no container do Nextcloud possa realizar qualquer alteração de permissão no diretório do volume durante a operação de implantação.

Execute a implantação novamente com `docker-compose up`. 

### Trusted Domains
Se ocorrer problemas de acesso via FQDN, finalize novamente a instância removendo os containers com `docker-compose down`, edite a configuração acrescentando a "trusted_domains" em `config/config.php` no volume `html` com o valor para o domínio e fqdn que definiu para o Nextcloud.

Exemplo:
```
  'trusted_domains' =>
  array (
    0 => 'nextcloud.lclhome.org',
    1 => 'lclhome.org ',
    2 => '192.168.5.120',
  ),
```
Suba o ambiente, `docker-compose up`.