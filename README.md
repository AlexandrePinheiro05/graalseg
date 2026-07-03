# GraalSeg — WordPress Completo

Site WordPress + WooCommerce replicando [GraalSeg](https://www.graalseg.com.br/).

## Estrutura do projeto

```
├── docker-compose.yml          # Ambiente local (Docker)
├── docker-compose.prod.yml     # Override para servidor
├── .env.example                # Variáveis de ambiente
├── wp-content/
│   ├── themes/graalseg/        # Tema + 85 cursos + imagens
│   └── mu-plugins/             # Plugin SMTP
├── scripts/
│   ├── setup-local.ps1         # Instalar e rodar no Windows
│   ├── setup-local.sh          # Instalar e rodar no Mac/Linux
│   ├── install-server.ps1      # Gerar pacote para FTP/hospedagem
│   └── bootstrap.sh            # Instala WP + WooCommerce + tema
└── config/
    ├── wp-config.template.php  # Config para servidor tradicional
    └── php/uploads.ini
```

---

## Rodar localmente (Docker) — recomendado

### Pré-requisito
Instale o [Docker Desktop](https://www.docker.com/products/docker-desktop/) e deixe-o aberto.

### Windows — um comando

Abra o PowerShell na pasta do projeto e execute:

```powershell
.\scripts\setup-local.ps1
```

### Mac / Linux

```bash
chmod +x scripts/setup-local.sh
./scripts/setup-local.sh
```

### Acessos após instalação

| O quê | URL |
|---|---|
| Site | http://localhost:8080 |
| Admin | http://localhost:8080/wp-admin |
| phpMyAdmin | http://localhost:8081 |
| Usuário | `admin` |
| Senha | `admin123` |

> Credenciais configuráveis em `.env`

### Comandos do dia a dia

```powershell
docker compose up -d          # iniciar
docker compose down           # parar
docker compose down -v        # reset total (apaga banco)
docker compose logs -f        # ver logs
```

---

## Deploy no servidor

Duas opções: **Docker** (AWS EC2, VPS) ou **hospedagem tradicional** (FTP/cPanel).

### Opção A — Servidor com Docker (AWS EC2, VPS)

1. Instale Docker no servidor
2. Envie o projeto (Git, SCP ou ZIP)
3. Configure produção:

```bash
cp .env.example .env
nano .env   # ajuste WP_URL, senhas fortes, WP_DEBUG=0
```

4. Suba em produção:

```bash
docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d
docker compose --profile tools run --rm wpcli
```

5. Aponte o domínio para o IP do servidor (porta 80)

---

### Opção B — Hospedagem tradicional (Hostinger, HostGator, etc.)

1. Gere o pacote completo no seu PC:

```powershell
.\scripts\install-server.ps1
```

2. Isso cria a pasta `dist\graalseg-server\` com WordPress core + tema + imagens

3. No painel da hospedagem:
   - Crie banco MySQL
   - Edite `wp-config.php` com dados do banco e URL do site
   - Gere salts em https://api.wordpress.org/secret-key/1.1/salt/

4. Faça upload de **todos** os arquivos de `dist\graalseg-server\` para `public_html/`

5. Acesse seu domínio, instale WooCommerce, ative tema **GraalSeg**

---

## O que já vem pronto

- Tema GraalSeg com layout igual ao site original
- 85 cursos importados automaticamente (preços, modalidades, imagens)
- Logo e banner oficiais
- Formulários (proposta, newsletter, depoimentos)
- Plugin SMTP (`Configurações → GraalSeg E-mail`)
- Páginas criadas automaticamente (Início, Contato, Sobre, etc.)

---

## Configurações pós-instalação

| Tarefa | Onde |
|---|---|
| Reimportar cursos | Aparência → Importar Produtos |
| Telefones / e-mail | Aparência → Personalizar → GraalSeg |
| SMTP | Configurações → GraalSeg E-mail |
| Pagamentos | WooCommerce → Configurações (fazer depois) |

---

## Requisitos

| Ambiente | PHP | MySQL | Outros |
|---|---|---|---|
| Local (Docker) | 8.2 | 8.0 | Docker Desktop |
| Servidor Docker | 8.2 | 8.0 | Docker + Compose |
| Hospedagem tradicional | 8.0+ | 5.7+ | Apache/Nginx, mod_rewrite |

---

## Subir no GitHub

### Importante

O **GitHub guarda o código**, mas **não hospeda** sites WordPress. Você não consegue abrir o site em `github.com/seu-usuario/repo`.

| O que o GitHub faz | O que você precisa para ver o site |
|---|---|
| Guardar e versionar o código | Servidor (AWS, Hostinger, VPS) ou Docker local |
| Deploy automático via Actions | Servidor com Docker + secrets configurados |

---

### Passo 1 — Instalar Git

1. Baixe: https://git-scm.com/download/win
2. Instale com as opções padrão
3. Reinicie o terminal / Cursor

Instale também o **GitHub CLI** (opcional): https://cli.github.com/

---

### Passo 2 — Criar repositório no GitHub

1. Acesse https://github.com/new
2. Nome: `graalseg` (ou outro)
3. Deixe **Private** se preferir
4. **Não** marque "Add README" (já temos arquivos)
5. Clique em **Create repository**

---

### Passo 3 — Enviar o projeto (primeira vez)

Abra o PowerShell na pasta do projeto:

```powershell
cd "C:\Users\MOTORISTA SOLINREL\Documents\Graalseg word"

git init
git add .
git commit -m "WordPress GraalSeg - tema, WooCommerce, Docker"
git branch -M main
git remote add origin https://github.com/SEU-USUARIO/graalseg.git
git push -u origin main
```

> Substitua `SEU-USUARIO` pelo seu usuário GitHub.  
> Na primeira vez, o GitHub pedirá login (navegador ou token).

**Token de acesso** (se pedir senha): GitHub → Settings → Developer settings → Personal access tokens → Generate new token (classic) → marque `repo`.

---

### Passo 4 — Atualizar o GitHub depois

Sempre que fizer alterações:

```powershell
git add .
git commit -m "Descreva o que mudou"
git push
```

Ou use: `.\scripts\git-push.ps1`

---

## Acessar o site na internet (a partir do GitHub)

O código no GitHub precisa ir para um **servidor**. Três caminhos:

### Caminho A — Deploy automático (recomendado)

1. Tenha um VPS (AWS EC2, Hostinger VPS, DigitalOcean)
2. No servidor, instale Docker e clone o repo:

```bash
git clone https://github.com/SEU-USUARIO/graalseg.git /var/www/graalseg
cd /var/www/graalseg
cp .env.example .env
nano .env   # WP_URL=https://seudominio.com.br, senhas fortes
docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d
docker compose --profile tools run --rm wpcli
```

3. No GitHub, configure **Secrets** (Settings → Secrets → Actions):
   - `SSH_HOST` — IP do servidor
   - `SSH_USER` — usuário SSH
   - `SSH_KEY` — chave privada SSH
   - `DEPLOY_PATH` — `/var/www/graalseg`

4. A cada `git push`, o workflow `.github/workflows/deploy.yml` atualiza o site automaticamente.

5. Aponte o domínio (DNS) para o IP do servidor.

**Site acessível em:** `https://seudominio.com.br` (não no github.com)

---

### Caminho B — Hospedagem tradicional (sem Docker)

1. No seu PC:

```powershell
.\scripts\install-server.ps1
```

2. Faça upload da pasta `dist\graalseg-server\` via FTP para o servidor
3. Mantenha o código-fonte no GitHub para versionamento

---

### Caminho C — Só local (desenvolvimento)

```powershell
.\scripts\setup-local.ps1
```

Acesse: http://localhost:8080

---

## O que NÃO vai para o GitHub

O `.gitignore` já exclui:
- Senhas (`.env`, `wp-config.php`)
- WordPress core baixado (`wp-admin/`, `dist/`)
- Uploads de usuários
- Banco de dados

**Nunca commite** arquivos `.env` com senhas reais.

