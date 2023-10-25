 # Certificados
 O Arquivo `openssl.conf` deverá ser alterado conforme suas necessidades, pois, lá, possui as variáveis que definem a localidade geográfica da sua organização, sua organização e a CN que responderá pelo seu domínio ou host, então, realmente, altere-o. 

 Execute a sequência de comandos abaixo para gerar sua Chave (.key):
 `openssl genpkey -algorithm RSA -out server.key`

 Após gerar a Key, obtenha a sua CSR:
 `openssl req -new -key server.key -out server.csr -config openssl.conf`

 E enfim, gere seu certificado:
 `openssl x509 -req -in server.csr -signkey server.key -out server.crt -extfile openssl.conf`

 Se tudo ocorreu bem até aqui, você deverá ter obtido os arquivos `server.csr`, `server.crt` e `server.key`, o que nos interessa aqui, são os arquivos `server.crt` e `server.key` que será usado pelo Nginx para prover https para acesso ao Nextcloud.