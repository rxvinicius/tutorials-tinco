# 02 — Perguntas e respostas sobre VPC

## Para que serve a VPC? E o Internet Gateway?

**VPC** — rede privada isolada na AWS. Tudo que a gente criar (banco, containers, load balancer) fica dentro dela. Sem VPC, nao tem onde colocar nada.

**Internet Gateway** — conexao entre a VPC e a internet. Sem ele, nada dentro da rede consegue ser acessado de fora (seus usuarios nao chegariam na API) e nada de dentro consegue sair.

## A gente tem subnets publicas para que os usuarios batam nas nossas APIs. Para que temos os privados?

Seguranca. O fluxo e assim:

1. Usuario acessa `api.tinco.app`
2. A requisicao chega no **Load Balancer** (subnet publica — acessivel pela internet)
3. O Load Balancer encaminha pra **API/containers** (subnet privada — nao acessivel pela internet)
4. A API consulta o **banco de dados** (subnet privada tambem)

Se os containers e o banco ficassem na subnet publica, qualquer pessoa na internet poderia tentar acessar diretamente — tentar conectar no banco, atacar a API sem passar pelo Load Balancer, etc.

Na subnet privada, o unico caminho de entrada e via Load Balancer. E uma camada de protecao. Ninguem de fora sabe nem que eles existem.

## O unico caminho de entrada e via Load Balancer, e a unica forma de se conectar ao Load Balancer e via nossa aplicacao? Dessa forma usuarios de fora nao conseguem se conectar ao nosso banco?

Quase. Invertendo um ponto:

O Load Balancer e **publico** — qualquer pessoa na internet pode se conectar nele (porta 80/443). Isso e intencional, porque os usuarios precisam acessar a API.

Mas o Load Balancer so encaminha o trafego pros **containers** (porta 3000). E os containers sao os unicos que conseguem falar com o **banco** (porta 5432).

Entao:

```
Qualquer pessoa  ->  Load Balancer  ->  Containers  ->  Banco
   (internet)        (publico)        (privado)     (privado)
```

Um usuario de fora **consegue** chegar no Load Balancer (e por consequencia na API), mas **nao consegue** chegar diretamente no banco. Quem controla exatamente quem fala com quem sao os **Security Groups**.

## So o Load Balancer se conecta com os containers?

Sim. Os Security Groups garantem isso. A regra diz: "containers so aceitam conexoes vindas do Load Balancer, rejeita todo o resto".
