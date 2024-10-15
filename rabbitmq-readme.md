# RabbitMQ com Docker (HTTP e HTTPS)

Este projeto configura um servidor RabbitMQ usando Docker, com suporte para conexões HTTP e HTTPS.

## Pré-requisitos

- Docker
- Docker Compose
- PowerShell (para Windows) ou bash (para Linux/MacOS)

## Configuração

1. Clone este repositório ou crie um novo diretório para o projeto.

2. Crie um arquivo `docker-compose.yml` com o seguinte conteúdo:

```yaml
version: '3.8'

services:
  rabbitmq:
    image: rabbitmq:3-management
    ports:
      - "5672:5672"   # AMQP protocol
      - "15672:15672" # HTTP management UI
      - "15671:15671" # HTTPS management UI
    environment:
      - RABBITMQ_DEFAULT_USER=user
      - RABBITMQ_DEFAULT_PASS=password
    volumes:
      - ./rabbitmq.conf:/etc/rabbitmq/rabbitmq.conf:ro
      - ./certs:/etc/rabbitmq/certs:ro
    deploy:
      resources:
        limits:
          memory: 2G
    healthcheck:
      test: ["CMD", "rabbitmq-diagnostics", "check_port_connectivity"]
      interval: 30s
      timeout: 10s
      retries: 3

volumes:
  rabbitmq_data:
```

3. Crie um arquivo `rabbitmq.conf` com o seguinte conteúdo:

```
listeners.tcp.default = 5672
management.tcp.port = 15672

listeners.ssl.default = 5671
ssl_options.cacertfile = /etc/rabbitmq/certs/cert.pem
ssl_options.certfile = /etc/rabbitmq/certs/cert.pem
ssl_options.keyfile = /etc/rabbitmq/certs/key.pem
ssl_options.verify = verify_peer
ssl_options.fail_if_no_peer_cert = false

management.ssl.port = 15671
management.ssl.cacertfile = /etc/rabbitmq/certs/cert.pem
management.ssl.certfile = /etc/rabbitmq/certs/cert.pem
management.ssl.keyfile = /etc/rabbitmq/certs/key.pem
```

4. Gere os certificados SSL:

Para Windows (PowerShell):

```powershell
# Criar diretório para os certificados
New-Item -ItemType Directory -Force -Path "certs"

# Gerar certificado autoassinado
$cert = New-SelfSignedCertificate -DnsName "localhost" -CertStoreLocation "Cert:\LocalMachine\My" -NotAfter (Get-Date).AddYears(1)

# Exportar o certificado público
$certPath = "certs\cert.pem"
Export-Certificate -Cert $cert -FilePath $certPath -Type CERT

# Exportar a chave privada
$keyPath = "certs\key.pem"
$pwd = ConvertTo-SecureString -String "password" -Force -AsPlainText
Export-PfxCertificate -Cert $cert -FilePath "certs\temp.pfx" -Password $pwd
openssl pkcs12 -in "certs\temp.pfx" -nocerts -out $keyPath -nodes -password pass:password

# Remover arquivo temporário
Remove-Item "certs\temp.pfx"

Write-Host "Certificados gerados com sucesso em $certPath e $keyPath"
```

Para Linux/MacOS (bash):

```bash
mkdir -p certs
openssl req -x509 -newkey rsa:4096 -keyout certs/key.pem -out certs/cert.pem -days 365 -nodes -subj "/CN=localhost"
```

## Executando o RabbitMQ

1. Inicie o RabbitMQ:

```
docker-compose up -d
```

2. Verifique se o contêiner está rodando:

```
docker-compose ps
```

3. Verifique os logs:

```
docker-compose logs rabbitmq
```

## Acessando o RabbitMQ

- Interface de gerenciamento HTTP: http://localhost:15672
- Interface de gerenciamento HTTPS: https://localhost:15671
- Conexão AMQP: localhost:5672
- Conexão AMQPS: localhost:5671

Use as credenciais definidas no `docker-compose.yml`:
- Usuário: user
- Senha: password

Nota: Ao acessar via HTTPS, você verá um aviso de segurança no navegador devido ao certificado autoassinado. Em um ambiente de produção, use certificados emitidos por uma autoridade certificadora confiável.

## Parando o RabbitMQ

Para parar o RabbitMQ:

```
docker-compose down
```

## Considerações de Segurança

- Altere o usuário e senha padrão em um ambiente de produção.
- Use certificados SSL emitidos por uma autoridade certificadora confiável em produção.
- Restrinja o acesso às portas do RabbitMQ conforme necessário.

