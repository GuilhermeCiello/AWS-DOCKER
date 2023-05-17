# AWS-DOCKER
Trabalho sobre AWS e Docker
# Implantação do Wordpress com Docker e AWS
# Descrição
O projeto tem como objetivo a criação de uma aplicação Wordpress em uma instância Amazon EC2 com ambiente baseado em Linux utilizando Docker-Compose junto do Wordpress em uma instância RDS com o banco de dados MYSQL. Também foram criados uma instância a mais para funcionar como Bastion Host para o acesso da instância principal, uma VPC para as instâncias, um Load Balancer para a organização do tráfego, o auto scaling group para tornar a aplicação escalável. O EFS foi usado para o armazenamento estático dos arquivos.

# Configuração da VPC
Acesso ao serviço VPC;
- Criação da VPC;
- Criação das subnets em duas zonas de disponibilidades diferentes, uma privada e uma pública na zona 1a e 1b;
- Criação do internet gateway e NAT gateway;
- Criação da tabela de rotas.

# Criação do Security Group
- Criação do security group para o Bastion Host com a porta 22 liberada para acesso SSH;
- Criação do security group porta HTTP, NFS, RDS e SSH.

# Criação das Instâncias
- Criação do Bastion Host e copiando a chave de acesso SSH para dentro dela;
- Criação da instância onde ficará a aplicação sem ip público, acesso feito através do Bastion Host.
# Instalação e Configuração do Docker na instância
- Criação de uma instância EC2 com ambiente AWS Linux 2.
- Durante a criação da instância, no campo "User data", foi adicionado o script abaixo para instalação automática do Docker e docker-compose.

#!/bin/bash
sudo yum update -y  
sudo yum install -y docker  
sudo systemctl start docker  
sudo curl -L https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose  
sudo chmod +x /usr/local/bin/docker-compose  
mv /usr/local/bin/docker-compose /bin/docker-compose  

# Upload no GitHub
- Criação de um repositório destinado para o trabalho "AWS-DOCKER";
- Inicialização do Git usando o comando “git init”;
- Comando “git status” para verificar se há arquivo(s) não adicionado(s) e comando “git add “[nome do arquivo]” para adicioná-lo(s);
- Configuração do git com os comandos “git config --global user.email “[email]” e “git config --global user.name “[nome]”;
- Commit através do comando “git commit -m “nome do commit”;
- Comando "git push" para enviar o conteúdo do repositório local para um repositório remoto.

# Configuração do EFS
- Criação do EFS  no console AWS;
- Acesso à instância ec2;
- montando o EFS dentro do diretório /etc/fstab  fs-07326fcfc0b337ad6.efs.us-east-1.amazonaws.com:/ home/ec2-user/efs nfs4 - nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport,_netdev 0 0;
- Comando sudo mount -a para montar.

# Configuração do RDS
- Acesso ao serviço RDS;
- Escolha do Banco MYSQL;
- Seleção da VPC e do security group criados para o projeto;
- Criação do nome do banco, usuário, e senha.

# Uso do Docker
Execução do script através do comando “docker-compose up -d” em segundo plano.
Script:
version: "3"
services:
  db:
    image: mysql:5.7
    restart: always
    environment:
      MYSQL_HOST: database-2.cdhzsu6mbxkf.us-east-1.rds.amazonaws.com
      MYSQL_ROOT_PASSWORD: senhabanco
      MYSQL_DATABASE: bancoDeDados
      MYSQL_USER: admin
      MYSQL_PASSWORD: senhabanco
  wordpress:
    depends_on:
      - db
    image: wordpress:latest
    restart: always
    ports:
      - "80:80"
    environment:
      - WORDPRESS_DB_HOST: database-2.cdhzsu6mbxkf.us-east-1.rds.amazonaws.com
      - WORDPRESS_DB_USER: admin
      - WORDPRESS_DB_PASSWORD: senhabanco
      - WORDPRESS_DB_NAME: bancoDeDados
    volumes:
      - "/home/ec2-user/efs:/var/www/html"
volumes:
  mysql: {}



# Criação do target group
- Seleção das instâncias que o LB irá atuar e habilitar o balanceamento de carga;
- Adição das intâncias no target group para associa-lás ao Load Balancer.

# Configuração do Load Balancer
- Acesso ao serviço EC2;
- Criação do Aplication Load Balancer:
- Configuração do Load Balancer (nome, VPC, security groups);
- Adição do target group criado anteriormente.


# Configuração do Auto Scaling
- Criação de uma AMI da instância usada;
- Criação de um launch configuration com base na AMI criada;
- Criação do grupo de Auto Scaling e associando ele ao target group.
