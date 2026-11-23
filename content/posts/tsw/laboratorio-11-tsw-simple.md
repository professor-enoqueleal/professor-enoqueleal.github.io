+++
title = "LAB 11 - Fazendo o deploy na AWS com Amazon EC2"
description = "Tutorial simples: criar EC2 com user-data que instala git e Java, clona o repositório e inicia a aplicação Spring Boot em background usando systemd." 
date = 2025-11-06
draft = true
author = "Enoque Leal"
tags = [ "aws", "ec2", "cloud-init", "spring-boot", "dockerless" ]
+++

## Objetivo


Neste laboratório vamos configurar o nosso projeto para fazer deploy em uma instância do **Amazon EC2**.
- Criar um instância do Amazon EC2 na AWS
- instala Git e Java (OpenJDK 17) no servidor EC2;
- clona o repositório **carstore-spring-boot** do GitHub;
- deixar a aplicação rodando no servidor par acesso remoto via navegador.

Ao final, quando a instância estiver pronta a aplicação estará em execução e acessível pela porta 8080.

---

## Resumo das etapas

1. Criar key pair e Security Group (abrir SSH e porta 8080).
2. Launch instance → colar o bloco **user-data** abaixo (cloud-init) na opção *Advanced -> User data*.
3. Após a inicialização, acessar **http://EC2_PUBLIC_IP:8080** para verificar a aplicação.

---

## Escolha da AMI e configurações iniciais

- AMI recomendada: **Ubuntu Server 22.04 LTS**.
- Instance type: **t3.micro** (ou conforme necessidade).
- Storage: 8–20 GB suficiente para teste.
- Security Group:
  - Porta 22 (SSH) — source: seu IP (recomendado);
  - Porta 8080 — source: 0.0.0.0/0 (ou restrinja por IP durante testes).
- Key pair: crie/baixe um par de chaves (.pem) e guarde-o com permissão 400.

---

## user-data (cloud-init) — cole este script ao criar a instância

O bloco abaixo é o *user-data* que automatiza a instalação, clone e criação do serviço systemd que roda a aplicação em background.

```bash
#!/bin/bash
set -xe

# Atualizar e instalar pacotes mínimos
apt-get update
apt-get upgrade -y
apt-get install -y git openjdk-17-jdk curl

# Variáveis (ajuste se necessário)
REPO_URL="https://github.com/professor-enoqueleal/dsw-projects"
# clonaremos o repositório no diretório /home/ubuntu/carstore
APP_DIR="/home/ubuntu/carstore"
SERVICE_NAME="carstore-app"

# Clonar ou atualizar o repositório como usuário ubuntu (mais seguro que como root)
if [ -d "$APP_DIR/.git" ]; then
  echo "Repositório já existe em $APP_DIR; fazendo git pull..."
  sudo -u ubuntu git -C "$APP_DIR" pull || true
else
  echo "Clonando o repositório em $APP_DIR (shallow clone)..."
  mkdir -p "$APP_DIR"
  sudo -u ubuntu git clone --depth 1 --branch main "$REPO_URL" "$APP_DIR"
fi

# Garantir permissões
chown -R ubuntu:ubuntu "$APP_DIR"

# Se o projeto estiver dentro de um subdiretório 'carstore-spring-boot', ajustamos o caminho
PROJECT_DIR="${APP_DIR}/carstore-spring-boot"

if [ -d "$PROJECT_DIR" ]; then
  echo "Projeto detectado em subdiretório: $PROJECT_DIR"
else
  # fallback: assumir que o repositório clonado é o próprio projeto
  PROJECT_DIR="$APP_DIR"
fi

# Tornar mvnw executável no subprojeto (se existir)
if [ -f "$PROJECT_DIR/mvnw" ]; then
  sudo -u ubuntu chmod +x "$PROJECT_DIR/mvnw"
fi

# Build do JAR (usa o Maven Wrapper presente no subprojeto)
echo "Compilando o projeto (gerando jar) em $PROJECT_DIR..."
sudo -u ubuntu bash -lc "cd '$PROJECT_DIR' && ./mvnw -B -ntp clean package -DskipTests"

# Verificar se o jar foi gerado
JAR_FILE=$(ls -1 $PROJECT_DIR/target/*.jar 2>/dev/null | head -n1 || true)
if [ -z "$JAR_FILE" ]; then
  echo "Erro: JAR não encontrado em $PROJECT_DIR/target/. Confira logs de build." >&2
  exit 1
fi

# Criar serviço systemd para executar o JAR gerado do subprojeto
cat > /etc/systemd/system/${SERVICE_NAME}.service <<'EOF'
[Unit]
Description=CarStore Spring Boot (run jar)
After=network.target

[Service]
User=ubuntu
WorkingDirectory=${PROJECT_DIR}
ExecStart=/usr/bin/java -jar ${PROJECT_DIR}/target/*.jar
SuccessExitStatus=143
Restart=on-failure
RestartSec=10
Environment=JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64

[Install]
WantedBy=multi-user.target
EOF

# Recarregar systemd, habilitar e iniciar o serviço
systemctl daemon-reload
systemctl enable ${SERVICE_NAME}.service
systemctl start ${SERVICE_NAME}.service

# Opcional: redirecionar logs (journalctl) — já gerenciado pelo systemd

```

Importante:
- Substitua **REPO_URL** pelo URL do repositório que contém o projeto (ex.: **https://github.com/SEU_USUARIO/carstore-spring-boot.git**). O repositório será clonado em `/home/ubuntu/carstore`.
- O script usa o **mvnw** (Maven Wrapper) presente no projeto; se o seu projeto não tiver **mvnw**, instale **maven** (apt-get install maven) e ajuste o **ExecStart** para `java -jar target/*.jar` (após build) ou para `mvn spring-boot:run` conforme preferir.

---

## Verificando se a aplicação está rodando

Após a instância iniciar (pode levar alguns minutos):

1. Pegue o IP público da instância (EC2 Console).
2. Acesse no navegador: http://EC2_PUBLIC_IP:8080.
3. Se precisar ver logs:

```bash
ssh -i ~/keys/minha-chave.pem ubuntu@EC2_PUBLIC_IP
# ver status do serviço
sudo systemctl status carstore-app.service
# ver logs
sudo journalctl -u carstore-app.service -f
```

---

## Observações e limitações

- Este tutorial prioriza simplicidade. Em produção prefira builds (jar) e execução via container (Docker) ou serviço de orquestração.
- Deixe a porta 22 restrita ao seu IP para reduzir exposição.
- Se o repo contém submódulos ou dependências privadas, ajuste o clone para usar chaves/credenciais adequadas.

---
