# GitLab vs GitHub - Comparativo para o projeto Tinco

## Comparativo de planos gratuitos

| Item                           | GitLab Free | GitHub Free                   |
| ------------------------------ | ----------- | ----------------------------- |
| Repos privados                 | Ilimitados  | Ilimitados                    |
| Minutos de pipeline/mes        | 400         | 2.000                         |
| Armazenamento de artifacts     | 5 GB        | 500 MB                        |
| Colaboradores (repos privados) | 5           | 3 (org) / ilimitado (pessoal) |
| Nome da pipeline               | GitLab CI   | GitHub Actions                |

## Comparativo de planos pagos

| Item                    | GitLab Premium | GitHub Team |
| ----------------------- | -------------- | ----------- |
| Preco                   | $29/user/mes   | $4/user/mes |
| Minutos de pipeline/mes | 10.000         | 3.000       |
| Colaboradores           | Ilimitados     | Ilimitados  |

## Estimativa de consumo de minutos (cenario Tinco)

Premissas:

- 3 repositorios (infra, backend, frontend)
- 2 desenvolvedores
- Cada deploy (build + push + invalidacao) leva cerca de 5-10 minutos

| Cenario            | Deploys/dia | Dias/mes | Minutos/mes | Cabe no GitHub Free?    |
| ------------------ | ----------- | -------- | ----------- | ----------------------- |
| Ritmo leve         | 3           | 22       | 66 - 132    | Sim (6% dos 2.000)      |
| Ritmo normal       | 10          | 22       | 220 - 440   | Sim (22% dos 2.000)     |
| Ritmo intenso      | 20          | 22       | 440 - 880   | Sim (44% dos 2.000)     |
| Limite GitLab Free | -           | -        | 400         | Estoura no ritmo normal |

## Limite de colaboradores

| Qtd de devs | GitHub Free     | Precisa pagar? |
| ----------- | --------------- | -------------- |
| Até 3       | Funciona        | Nao            |
| 4+          | Precisa do Team | $4/user/mes    |

## Conclusao

Para um time de 2 devs com 3 repositorios, o plano gratuito do GitHub atende com folga.
O GitLab Free pode estourar os minutos em meses mais intensos de desenvolvimento.
Se o time crescer para 4+ pessoas, o GitHub Team ($4/user/mes) e 7x mais barato que o GitLab Premium ($29/user/mes).
