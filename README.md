# Atividade-Docker-AWS
# **Documentação Completa do Projeto - WordPress com AWS e Docker**

## **1️⃣ Introdução**
Este documento detalha todo o processo de configuração do ambiente AWS para hospedar uma aplicação WordPress utilizando **Docker**, **Amazon EC2**, **Amazon RDS**, **Amazon EFS** e **Security Groups**.

## **2️⃣ Tecnologias Utilizadas**
- **Amazon Web Services (AWS)**
- **Amazon EC2** (Elastic Compute Cloud)
- **Amazon RDS** (Relational Database Service)
- **Amazon EFS** (Elastic File System)
- **Docker**
- **Docker Compose**
- **MySQL**
- **WordPress**
- **Git e GitHub**
- **Linux (Git-Bash)**
- **NFS (Network File System)**

## **3️⃣ Configuração do AWS Console**

### **3.1 Criar Instância EC2**
1. Acesse o **AWS Console** e navegue até o **EC2**.
2. Clique em **Launch Instance**.
3. Escolha a imagem **Ubuntu 22.04 LTS**.
4. Escolha o tipo da instância: `t2.micro` (grátis no Free Tier da AWS).
5. Configure:
   - Defina um nome para a instância (`WordPress-EC2`).
   - Escolha a VPC padrão.
   - Configure o Security Group (detalhes abaixo).
   - Gere e baixe um **par de chaves SSH** para acesso remoto.
6. Clique em **Launch**.

### **3.2 Configurar Security Groups**
Crie um **Security Group** e configure as regras de acesso:

| Tipo       | Protocolo | Porta | Origem           | Descrição        |
|------------|----------|-------|------------------|----------------|
| SSH        | TCP      | 22    | Seu IP (My IP)   | Acesso SSH     |
| HTTP       | TCP      | 80    | 0.0.0.0/0        | Acesso Web     |
| HTTPS      | TCP      | 443   | 0.0.0.0/0        | Acesso Web SSL |
| MySQL/Aurora | TCP    | 3306  | IP do EC2        | Conexão MySQL   |


## **4️⃣ Configurar o Banco de Dados (RDS MySQL)**

### **4.1 Criar Banco de Dados RDS**
1. No **AWS Console**, navegue até **Amazon RDS**.
2. Clique em **Create Database**.
3. Escolha **MySQL 8.0**.
4. Modo: **Standard Create**.
5. Selecione `db.t3.micro` (Free Tier).
6. Configure:
   - **Database Name:** 
   - **Username:** 
   - **Password:** 
7. Escolha a mesma **VPC do EC2**.
8. Clique em **Create Database**.

### **4.2 Configurar Acesso ao RDS**
1. Adicione o **Security Group do EC2** ao RDS.
2. Pegue o **Endpoint** do banco e teste a conexão:
   ```bash
   mysql -h database-projeto.clguk20sen5s.us-east-1.rds.amazonaws.com -u admin -p
   ```

## **5️⃣ Configurar Amazon EFS para Arquivos Estáticos**

1. No **AWS Console**, acesse **EFS** e clique em **Create File System**.
2. Escolha a mesma **VPC do EC2**.
3. Configure permissões de acesso para permitir conexão com EC2.
4. Pegue o **File System ID** e monte no EC2:
   ```bash
   sudo mount -t nfs4 -o nfsvers=4.1 fs-0018990c9972e8485.efs.us-east-1.amazonaws.com:/ /mnt/efs
   ```

## **6️⃣ Configuração do Docker e Docker Compose**

### **6.1 Instalar Docker no EC2**
```bash
sudo apt update && sudo apt install -y docker.io
sudo systemctl start docker
sudo systemctl enable docker
```

### **6.2 Instalar Docker Compose**
```bash
sudo apt install -y docker-compose
```

## **7️⃣ Criar o Arquivo docker-compose.yml**
Crie um arquivo `docker-compose.yml` com o seguinte conteúdo:

```yaml
version: '3.1'

services:
  db:
    image: mysql:5.7
    restart: always
    environment:
      MYSQL_DATABASE: wordpress
      MYSQL_USER: admin
      MYSQL_PASSWORD: senha123
      MYSQL_ROOT_PASSWORD: senha123
    volumes:
      - db_data:/var/lib/mysql

  wordpress:
    image: wordpress
    restart: always
    ports:
      - "80:80"
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: 
      WORDPRESS_DB_PASSWORD: 
      WORDPRESS_DB_NAME: wordpress
    volumes:
      - wp_data:/var/www/html

volumes:
  db_data:
  wp_data:
    driver: local
    driver_opts:
      type: "nfs"
      o: "addr=fs-0018990c9972e8485.efs.us-east-1.amazonaws.com,rw"
      device: ":/"
```

## **8️⃣ Executar os Containers**

```bash
docker-compose up -d
```

Verifique se os containers estão rodando:
```bash
docker ps
```

Acesse o WordPress pelo navegador em:
```
http://52.91.59.95/:80
```

## **9️⃣ Configurar o Git e Enviar o Projeto para o GitHub**

### **9.1 Configurar SSH para GitHub**
```bash
ssh-keygen -t ed25519 -C "xafullt@example.com"
ssh-add ~/.ssh/id_ed25519
cat ~/.ssh/id_ed25519.pub
```
Adicione essa chave ao GitHub.

### **9.2 Enviar arquivos para o GitHub**
```bash
git init
git remote add origin git@github.com:thiagodemorais87/Atividade-Docker-AWS.git
git add .
git commit -m "Projeto WordPress com Docker e AWS"
git push -u origin main
```

## **🔟 Conclusão**
Este documento cobre toda a configuração do ambiente AWS para rodar o WordPress utilizando Docker. Se precisar de ajustes, consulte os logs dos containers:

```bash
docker logs -f wordpress
```

Agora seu site WordPress está rodando na AWS! 🚀


Entre em contato: thiagomgoncalves87@gmail.com
