# 01 — VPC e Rede (network.tf)

## O que esse arquivo cria?

Uma rede privada na AWS onde todos os outros recursos vão morar.

## Conceitos

### VPC (Virtual Private Cloud)
Rede privada isolada na AWS. Tudo que a gente criar fica dentro dela.
- CIDR `10.0.0.0/16` = 65.536 endereços IP disponíveis
- `enable_dns_support` e `enable_dns_hostnames` permitem que os serviços se encontrem por nome dentro da rede (ex: o container consegue achar o banco por um hostname em vez de decorar um IP)

### Subnets
Divisões dentro da VPC. Cada uma tem uma faixa de IPs e fica em um datacenter (zona de disponibilidade) específico.

**Públicas** (`10.0.1.0/24` e `10.0.2.0/24`):
- Acessíveis pela internet
- Aqui fica o Load Balancer (a "porta de entrada" da API)
- `map_public_ip_on_launch = true` — recursos criados aqui ganham IP público automaticamente

**Privadas** (`10.0.10.0/24` e `10.0.20.0/24`):
- SEM acesso direto da internet
- Aqui ficam os containers (ECS) e o banco de dados (RDS)
- Mais seguro — ninguém de fora acessa direto

**Por que 2 de cada?** A AWS exige pelo menos 2 subnets em zonas de disponibilidade diferentes pra serviços como RDS e ALB. É pra redundância — se um datacenter cair, o outro segura.

### Internet Gateway
Conexão entre a VPC e a internet. Sem ele, nada dentro da VPC consegue ser acessado de fora, e nada de dentro consegue sair.

### NAT Gateway
Permite que recursos na subnet **privada** acessem a internet (ex: enviar e-mail, chamar API externa), mas sem que ninguém de fora consiga acessar eles. Tráfego só sai, não entra.
- Precisa de um **Elastic IP** (IP fixo) pra funcionar
- Fica posicionado na subnet pública (porque ele precisa de acesso à internet pra funcionar como intermediário)

### Tabelas de rota (Route Tables)
Regras de tráfego. Dizem pra onde o tráfego vai quando um recurso quer acessar a internet:
- Subnet pública → tráfego sai pelo **Internet Gateway** (acesso direto)
- Subnet privada → tráfego sai pelo **NAT Gateway** (acesso indireto, só saída)
- `0.0.0.0/0` = "qualquer destino fora da VPC" (basicamente, a internet)

### Associações (Route Table Associations)
Conectam cada subnet à sua tabela de rota. Sem isso, a subnet não sabe por onde rotear o tráfego.

## Diagrama

```
Internet
    │
    ▼
Internet Gateway
    │
    ├── Subnet pública A (10.0.1.0/24) ── Load Balancer
    ├── Subnet pública B (10.0.2.0/24) ── Load Balancer
    │
    ▼
NAT Gateway (na subnet pública A)
    │
    ├── Subnet privada A (10.0.10.0/24) ── Containers (ECS) + Banco (RDS)
    └── Subnet privada B (10.0.20.0/24) ── Containers (ECS) + Banco (RDS)
```

## Referências
- [AWS VPC Documentation](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html)
- [Terraform aws_vpc](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/vpc)
