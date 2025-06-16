# N8N-Backup

Backup Automático de Workflows do N8N para o GitHub:

Este workflow deve fazer um backup diário dos seus fluxos no N8N direto para um repositório no GitHub, criando ou atualizando arquivos JSON.

---

Visão Geral

1. Gatilho Agendado:  
   O fluxo dispara automaticamente todos os dias em um horário pré-configurado.

2. Listar Backups Existentes:  
   Conecta ao repositório GitHub e recupera a lista de arquivos já salvos.

3. Recuperar Workflows n8n:  
   Busca todos os workflows atuais da sua instância n8n via API REST.

4. Processar e Preparar:  
   - Converte cada workflow em JSON.  
   - Codifica o JSON em Base64 (requisito da API do GitHub).

5. Confirmar no GitHub:  
   Para cada workflow:
   - Gera um nome de arquivo padronizado:  
     ```
     <workflow-name>-<tag-primária>.json
     ```
   - Verifica se já existe no repositório:
     - Se existir: atualiza o arquivo usando o SHA obtido.  
     - Se for novo: cria o arquivo.  
   - Usa mensagem de commit com data e hora para histórico claro.

> Resultado: você mantém sempre uma cópia versionada e atualizada de todos os seus workflows n8n no GitHub.

---

Pré-requisitos

Antes de começar, tenha em mãos:

- Uma instância n8n ativa (self-hosted ou n8n.cloud)  
- Conta GitHub e repositório dedicado aos backups  
- GitHub Personal Access Token** com escopo `repo` (read-write)  
- API Key n8n para sua instância (definida em Settings → n8n API)

---

Etapas de Configuração

1. Importar o modelo 
   - No n8n, clique em "Import from Clipboard" e cole o JSON do workflow.

2. Credenciais n8n 
   - Localize o nó "Recuperar workflows".  
   - Selecione / crie credencial do tipo "n8n API":  
     - Base URL: `https://<seu-host>/api/v1`  
     - API Key: sua chave configurada.

3. Credenciais GitHub  
   - Localize o nó "Listar arquivos do repositório" (e depois "Update file" / "Upload file").  
   - Selecione / crie credencial **GitHub** (OAuth2 / Personal Access Token).  
   - Insira seu token GitHub.

4. Detalhes do Repositório  
   Em cada nó GitHub (“List files”, “Get file”, “Update file”, “Upload file”):
   - Owner: seu usuário ou organização  
   - Repository: `N8N-Backup` (exemplo)  
   - Branch: `main` (ou seu branch padrão)  
   - Path: (opcional) pasta dentro do repo, ex. `n8n_backups/` — deixe em branco para raiz.

5. Ajustar Programação (opcional)  
   - Abra o nó "Schedule Trigger" e altere horário/frequência conforme desejar.

6. "Ativar Workflow"  
   - Salve e ative. O backup começará a rodar conforme a agenda.

---

Descrição dos Nós

| Nó                             | Tipo                                              | Objetivo                                                       |
|--------------------------------|---------------------------------------------------|----------------------------------------------------------------|
| Schedule Trigger               | n8n-nodes-base.scheduleTrigger                    | Inicia o workflow conforme horário agendado.                   |
| List files from repo           | n8n-nodes-base.github (get: repository)           | Lista todos os arquivos no repositório de backup.              |
| Aggregate                      | n8n-nodes-base.aggregate                          | Consolida nomes de arquivo em um único item.                   |
| Retrieve workflows             | n8n-nodes-base.n8n (getAll: workflow)             | Busca todos os workflows da instância n8n.                     |
| Json file                      | n8n-nodes-base.convertToFile                      | Converte dados em arquivo JSON.                                |
| To base64                      | n8n-nodes-base.extractFromFile                    | Transforma JSON em string Base64.                              |
| Commit date & file name        | n8n-nodes-base.set                                | Gera `commitDate` e `fileName` padronizados.                   |
| Get file                       | n8n-nodes-base.github (get: File)                 | Tenta buscar arquivo existente e retorna `sha` (Continue on Fail). |
| If                             | n8n-nodes-base.if                                 | Verifica se `sha` existe (`Is Not Empty`).                     |
| Update file                    | n8n-nodes-base.github (edit: File)                | Atualiza arquivo existente pelo `sha`.                         |
| Upload file                    | n8n-nodes-base.github (create: File)              | Cria novo arquivo quando não existe.                           |

---

Personalização

- Pasta de Backup: defina `Path` nos nós GitHub para agrupar em subpastas.  
- Nome do Arquivo: altere a expressão em "Commit date & file name" para outro padrão.  
- Mensagem de Commit: edite a mensagem nos nós “Update file” e “Upload file”.  
- Filtrar Workflows: insira um nó "Filter" após “Retrieve workflows” para incluir apenas certos tags/names.  
- Tratamento de Erros: adicione ramificações de erro para notificações (ex.: nó “Error Trigger”).  
- Frequência de Backup: ajuste o **Schedule Trigger** para horários, dias ou intervalos diferentes.
 
> Data de criação: 16-06-2025  
