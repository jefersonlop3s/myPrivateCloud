# Freeipa Notes


## Teste de conectividade em portas

**Exemplo** de teste se uma determinada porta está aberta ou fechada em um determinado host quando o não for possível o uso do Nmap:
``` 
echo > /dev/udp/$LoadBalancerFreeipa/$port
echo > /dev/tcp/$LoadBalancerFreeipa/$port

```
Quanda não há conectividade na porta testada, teremos um `connect: Connection refused`.

Na prática:
```
echo > /dev/udp/192.168.5.6/53 && echo "Opened"
```

## Referências: 
1. https://hub.docker.com/r/freeipa/freeipa-server
2. https://github.com/freeipa/freeipa-container
    * https://github.com/freeipa/freeipa-container/blob/master/freeipa-server-openshift.json
    * https://github.com/freeipa/freeipa-container/blob/master/freeipa-server-openshift-volume.json
    * https://github.com/freeipa/freeipa-container/blob/master/freeipa-server-openshift-image.json
3. Dica de Teste de Conectividade: https://www.vivaolinux.com.br/dica/Verificar-se-determinada-porta-esta-aberta-com-o-comando-echo-funciona-remoto 

