# Setup do n8n em Docker para VPS (Pronto para Produção)

Este repositório contém a configuração ideal para rodar o **n8n** na sua VPS utilizando **Docker Compose**, banco de dados **PostgreSQL** (para melhor performance) e o **Caddy** como proxy reverso com geração e renovação automática de certificado SSL (HTTPS).

## 🚀 Estrutura dos Serviços
- **n8n**: Plataforma de automação de fluxo de trabalho.
- **PostgreSQL**: Banco de dados relacional robusto para salvar fluxos, execuções e credenciais.
- **Caddy**: Servidor web leve e seguro que gerencia o SSL (Let's Encrypt / ZeroSSL) automaticamente e repassa o tráfego de forma segura para o n8n.

---

## 🛠️ Pré-requisitos na VPS

1. **Docker e Docker Compose instalados**:
   Caso não tenha instalado na VPS (Ubuntu/Debian), execute:
   ```bash
   sudo apt update
   sudo apt install -y docker.io docker-compose-v2
   sudo systemctl enable --now docker
   ```
2. **Apontamento de DNS (Obrigatório para o SSL)**:
   No seu gerenciador de domínio (ex: Cloudflare, Registro.br, Hostgator), crie um **registro do tipo A** apontando para o IP público da sua VPS:
   - **Nome**: `n8n` (ou o subdomínio que preferir)
   - **Destino (Aponta para)**: `IP_DA_SUA_VPS`
   - *Nota se usar Cloudflare*: Deixe a nuvem de proxy desativada (DNS Only) inicialmente para que o Let's Encrypt consiga validar o certificado HTTP-01 facilmente.
3. **Liberar Portas no Firewall da VPS**:
   Certifique-se de que as portas **80** e **443** (TCP) estão abertas no firewall da sua VPS:
   ```bash
   sudo ufw allow 80/tcp
   sudo ufw allow 443/tcp
   sudo ufw allow 443/udp
   ```

---

## 📦 Como Instalar e Rodar

### 1. Clonar ou Copiar os Arquivos para a VPS
Crie uma pasta na sua VPS (ex: `/opt/n8n`) e copie os seguintes arquivos para lá:
- `docker-compose.yml`
- `Caddyfile`
- `.env`

### 2. Configurar as Variáveis de Ambiente
Abra o arquivo `.env` na VPS e configure os valores reais:
```bash
nano .env
```
Campos principais a alterar:
- **`N8N_HOST`**: Coloque o seu subdomínio completo (ex: `n8n.seudominio.com`).
- **`WEBHOOK_URL`**: Deve ser `https://n8n.seudominio.com/` (não esqueça da barra no final).
- **`SSL_EMAIL`**: Seu e-mail de contato para notificações do certificado de SSL.
- **`POSTGRES_PASSWORD`** e **`N8N_ENCRYPTION_KEY`**: Chaves seguras já foram geradas por padrão no seu `.env`, mas você pode alterá-las se desejar.

*(Salve o arquivo pressionando `Ctrl + O`, `Enter` e depois `Ctrl + X` para sair do nano)*

### 3. Iniciar o n8n
Com as configurações salvas, suba os contêineres em segundo plano (modo daemon):
```bash
docker compose up -d
```

---

## 🔍 Comandos Úteis de Gerenciamento

- **Verificar se os contêineres estão rodando**:
  ```bash
  docker compose ps
  ```
- **Ver os logs em tempo real (muito útil para depurar SSL ou conexão de banco)**:
  ```bash
  docker compose logs -f --tail=100
  ```
- **Reiniciar os serviços**:
  ```bash
  docker compose restart
  ```
- **Parar o n8n**:
  ```bash
  docker compose down
  ```

---

## 💾 Backup e Segurança

Os dados do n8n e do banco de dados são persistidos em volumes gerenciados pelo Docker para evitar perdas ao atualizar as imagens:
- **Banco de dados**: Volume `n8n_postgres_storage` localizado em `/var/lib/docker/volumes/n8n_postgres_storage/_data` na VPS.
- **Arquivos locais do n8n**: Volume `n8n_storage_data` localizado em `/var/lib/docker/volumes/n8n_storage_data/_data`.

### Como fazer backup rápido do banco de dados PostgreSQL:
```bash
docker exec -t n8n_postgres_db pg_dumpall -c -U n8n_postgres_admin > backup_n8n_$(date +%F).sql
```
