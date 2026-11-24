# Configuração do MCP — GitHub no Kiro

Guia para configurar o acesso do Kiro aos repositórios da Tinco no GitHub.

## O que é o MCP?

O MCP (Model Context Protocol) permite que o Kiro se conecte a serviços externos como o GitHub. Com ele configurado, o Kiro consegue ler PRs, fazer code review, buscar arquivos e interagir diretamente com os repositórios da organização `tinco-app`.

## Passo 1 — Gerar o Token do GitHub

1. Acesse: [github.com/settings/tokens](https://github.com/settings/tokens)
2. Clique em **"Generate new token"** → **"Generate new token (classic)"**
3. Preencha:
   - **Note**: `Kiro MCP`
   - **Expiration**: escolha o prazo que preferir (90 dias é um bom padrão)
   - **Scopes**: marque `repo` (acesso completo a repositórios privados)
4. Clique em **"Generate token"**
5. **Copie o token gerado** (começa com `ghp_...`) — ele só aparece uma vez

## Passo 2 — Criar o arquivo de configuração

Crie o arquivo `~/.kiro/settings/mcp.json` na sua home. Esse caminho faz a configuração valer para **qualquer projeto** que você abrir no Kiro.

```bash
mkdir -p ~/.kiro/settings
```

Crie o arquivo `~/.kiro/settings/mcp.json` com o seguinte conteúdo:

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "COLE_SEU_TOKEN_AQUI"
      }
    }
  }
}
```

Substitua `COLE_SEU_TOKEN_AQUI` pelo token que você copiou no passo anterior.

## Passo 3 — Verificar

1. Abra (ou reabra) o Kiro em qualquer repositório da Tinco
2. Peça algo como: *"me mostra os PRs abertos do napoleon"*
3. Se o Kiro conseguir listar os PRs, está funcionando

## Observações

- O token é **pessoal** — cada dev cria o seu
- O arquivo `mcp.json` **não vai pro git** (está no `.gitignore` dos repositórios)
- Se o token expirar, basta gerar um novo e atualizar o arquivo
- Requer **Node.js** instalado (o `npx` precisa estar disponível no terminal)
