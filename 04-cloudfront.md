# 04 — CloudFront (CDN)

## O que e CloudFront?

CloudFront e uma CDN (Content Delivery Network) da AWS. Ele distribui os arquivos do seu site em servidores espalhados pelo mundo para que os usuarios acessem de um servidor proximo, em vez de ir buscar nos EUA toda vez.

## Como funciona

A AWS tem mais de 450 servidores espalhados pelo mundo, chamados de **edge locations**. Quando voce cria uma distribuicao CloudFront, ela automaticamente fica disponivel em todos esses locais.

### Fluxo de uma requisicao

1. Usuario no Brasil acessa `painel.tinco.app`
2. O DNS resolve para a edge location mais proxima (Sao Paulo)
3. **Primeira vez**: o servidor de Sao Paulo nao tem o arquivo -> vai buscar no S3 (us-east-1) -> guarda em cache -> entrega pro usuario
4. **A partir dai**: o servidor de Sao Paulo ja tem o arquivo no cache -> entrega direto, sem ir pro S3

O CloudFront sabe qual servidor usar pelo **DNS**. Quando o navegador resolve o dominio, a AWS retorna o IP do servidor mais proximo geograficamente. Voce nao configura nada disso — é automatico.

### Expiracao do cache

O cache expira baseado no `default_ttl` que configuramos (3600 segundos = 1 hora). Depois de 1 hora, o CloudFront busca de novo no S3. Quando a pipeline faz um novo deploy, ela roda uma "invalidacao" que limpa o cache de todos os edge locations, forcando o CloudFront a buscar a versao nova.

## Com CDN vs sem CDN

|                        | Com CloudFront                     | Sem CloudFront (S3 direto)     |
| ---------------------- | ---------------------------------- | ------------------------------ |
| Latencia (Brasil)      | ~20ms (servidor em SP)             | ~150-200ms (servidor nos EUA)  |
| HTTPS                  | Sim, gratis                        | Nao (S3 sozinho so serve HTTP) |
| Cache                  | Sim (reduz custo e carga no S3)    | Nao                            |
| Custo de transferencia | Menor (cache reduz requests ao S3) | Maior                          |

## Porque demora 3-5 minutos pra criar

Quando voce roda `terraform apply`, a AWS precisa propagar a configuracao da distribuicao para todos os 450+ edge locations no mundo. Cada servidor precisa receber as regras de cache, o certificado SSL, as configuracoes de origin, etc. Essa propagação global e o que leva tempo.

## Custom error responses (SPA)

Configuramos o CloudFront para retornar `index.html` quando recebe erros 403 ou 404. Isso e necessario porque o React usa rotas no navegador (React Router). Quando alguem acessa `painel.tinco.app/collaborators`, o S3 nao tem um arquivo chamado `/collaborators` — daria 404. Com essa config, o CloudFront retorna o `index.html` e o React Router cuida de mostrar a pagina certa.

## Origin Access Control (OAC)

Garante que o bucket S3 so aceita requisições vindas do CloudFront. Ninguém consegue acessar os arquivos diretamente pela URL do S3. Todas as requisições passam obrigatoriamente pelo CloudFront.
