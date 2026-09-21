# 03 — ALB (Application Load Balancer)

## O que e o ALB?

E o ponto de entrada da internet pra API. Recebe as requisicoes dos usuarios e encaminha pros containers no ECS.

## Diferenca entre ALB e ECS

| | ALB | ECS |
|---|---|---|
| O que e | Porteiro | Funcionario |
| Onde fica | Subnet publica (acessivel da internet) | Subnet privada (protegido) |
| O que faz | Recebe requisicoes e repassa | Roda o codigo da API |
| Se nao existir | Ninguem acessa a API | Nao tem API rodando |

## Diagrama completo (do usuario ate o banco)

```
+------------------------------------------------------------------+
|                        INTERNET                                  |
|                                                                  |
|   Usuario digita: https://api.tinco.app/collaborators            |
+----------------------------+-------------------------------------+
                             |
                             v
+------------------------------------------------------------------+
|                  SUBNET PUBLICA                                  |
|                                                                  |
|   +-------------------------------------+                        |
|   |         ALB (Load Balancer)         |                        |
|   |                                     |                        |
|   |  Porta 80  -> recebe a requisicao   |                        |
|   |  Verifica  -> health check ok?      |                        |
|   |  Encaminha -> pro Target Group      |                        |
|   +------------------+------------------+                        |
|                      |                                           |
+----------------------+-------------------------------------------+
                       | porta 3000
                       v
+------------------------------------------------------------------+
|                  SUBNET PRIVADA                                  |
|                                                                  |
|   +-------------------------------------+                        |
|   |      ECS (Container Node.js)        |                        |
|   |                                     |                        |
|   |  Recebe a requisicao na porta 3000  |                        |
|   |  Executa: GET /collaborators        |                        |
|   |  Consulta o banco ----------------------+                    |
|   +-------------------------------------+   |                    |
|                                             | porta 5432         |
|   +-----------------------------------------v---+                |
|   |      RDS (Banco PostgreSQL)                  |               |
|   |                                              |               |
|   |  SELECT * FROM collaborators                 |               |
|   |  Retorna os dados -> container -> ALB ->     |               |
|   |  -> usuario ve a resposta JSON               |               |
|   +----------------------------------------------+               |
|                                                                  |
+------------------------------------------------------------------+
```

## O que e o Health Check?

O ALB bate periodicamente no endpoint /status da API pra verificar se o container esta saudavel. Se o container nao responder 3 vezes seguidas, o ALB para de mandar trafego pra ele e o ECS sobe um novo.

Configuracao do health check:
- Path: /status (endpoint da API napoleon)
- Intervalo: a cada 30 segundos
- Timeout: 5 segundos pra responder
- Healthy threshold: 3 respostas OK seguidas = saudavel
- Unhealthy threshold: 3 falhas seguidas = nao saudavel
- Matcher: aceita respostas com status HTTP 200 a 399

## O que sao os sub-recursos do ALB?

- ALB: o load balancer em si
- Target Group: grupo de destino onde os containers se registram
- Listener: escuta numa porta especifica e decide o que fazer (encaminhar pro target group, redirecionar, etc)

## Seguranca do ALB

O ALB nao faz rate limiting nem validacao de seguranca avancada. Ele so roteia trafego. Pra rate limiting, precisaria de AWS WAF (Web Application Firewall) — nao incluimos por enquanto porque tem custo extra e pro volume do projeto nao e necessario.

O que o ALB faz de seguranca:
- So aceita trafego nas portas 80 e 443 (via security group alb-sg)
- Redireciona HTTP pra HTTPS (depois que configurarmos o certificado SSL)
- Health check garante que trafego so vai pra containers saudaveis
