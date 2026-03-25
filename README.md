# Kestra Git Integration🚀

Este repositório contém testes de integração entre o orquestrador **Kestra** e o **GitHub**. O objetivo é validar o versionamento automático de fluxos (Flows) e Namespaces diretamente para um repositório remoto.

---

## 🛠️ Passo a Passo para Configuração

Siga as etapas abaixo para permitir que o Kestra realize commits e pushes automaticamente.

### 1. Criar o Repositório
Crie o repositório no GitHub. Certifique-se de definir a branch inicial (ex: `main` ou `develop`) antes de iniciar o primeiro push pelo Kestra.

### 2. Gerar Token de Acesso (Classic)
Vá ao seu GitHub: **Settings > Developer Settings > Personal access tokens (classic)**.
* link direto: https://github.com/settings/tokens
* Gere um novo token com a permissão `repo`.
* Copie o token gerado (ex: `ghp_fPbsIVw...`).

### 3. Configurar o Secret no Docker Compose do Kestra
O Kestra utiliza chaves criptografadas para segurança. Você deve converter seu token para **Base64** antes de adicioná-lo ao seu ambiente Docker.

No seu terminal (Ubuntu/Linux), execute:
```bash
echo -n "ghp_seu_token_aqui" | base64
```

Pegue o resultado gerado e adicione no seu arquivo docker-compose.yml na seção de variáveis de ambiente do Kestra:
```YAML
kestra:
  environment:
    SECRET_GITHUB_ACCESS_TOKEN: "RESULTADO_DO_BASE64_AQUI"
```
### 4. Criar e Executar o Flow de Backup
Crie um novo Flow no Kestra com o código abaixo. Este script varre o namespace company e envia todos os flows encontrados para a pasta flows/ no GitHub.

Flow Principal de Backup:

```YAML
id: push_flow_to_git
namespace: system
description: Subir flows e namespaces para um repositório remoto

tasks:
  - id: commit_and_push
    type: io.kestra.plugin.git.PushFlows

    # Credenciais e Repositório
    username: lauanderson-rael
    password: "{{ secret('GITHUB_ACCESS_TOKEN') }}"
    url: [https://github.com/lauanderson-rael/flow-example](https://github.com/lauanderson-rael/flow-example)
    branch: develop

    # Configuração de Origem
    sourceNamespace: company
    includeChildNamespaces: true  # Garante que pegará company.team, etc.
    
    # Configuração de Destino no Git
    gitDirectory: flows
    commitMessage: "changes to kestra flows - {{ now() }}"

    # Controle de Execução
    dryRun: false # Mudar para true se quiser apenas testar sem enviar
```
