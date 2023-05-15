# AWS-DOCKER
Trabalho sobre AWS e Docker
# Implantação do Wordpress com Docker e AWS
# Descrição
O projeto tem como objetivo a criação de uma aplicação Wordpress em uma instância Amazon EC2 com ambiente baseado em Linux utilizando Docker-Compose junto de um sistema livre para gestão de conteudo na internet, o Wordpress em uma instância RDS com o banco de dados MYSQL. Além da instância principal, o Docker-Compose e o Wordpress, também foram necessários criar uma instância a mais para funcionar como Bastion Host para o acesso da principal, uma VPC para as instâncias e um Load Balancer para a organização do tráfego. Também foi usado EFS para o armazenamento estático dos arquivos.
Pré-requisitos
Possuir uma conta na AWS com permissões suficientes para criar e configurar os serviços mencionados.
Conhecimento básico de Docker e containers.
Conhecimento básico de Git e repositórios.

# Configuração da VPC
Acesso ao serviço VPC;
Criação da VPC;
Criação das subnets em duas zonas de disponibilidades diferentes, uma privada e uma pública na zona 1a e 1b;
Criação do internet gateway e NAT gateway;
Criação da tabela de rotas.
Instalação e Configuração do Docker na instância
Criação de uma instância EC2 com ambiente AWS Linux 2.
Durante a criação da instância, no campo "User data", foi adicionado o script abaixo para instalação automática do Docker:

#!/bin/bash
sudo yum update -y
sudo yum install -y docker
sudo systemctl  docker start
sudo curl -L https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
mv /usr/local/bin/docker-compose /bin/docker-compose

# Upload no GitHub
Criação de um repositório destinado para o trabalho.
Download do Git na instância através do comando “sudo yum install git”.
Inicialização do Git usando o comando “git init”.
Comando “git status” para verificar se há arquivo(s) não adicionado(s) e comando “git add “[nome do arquivo]”” para adicioná-lo(s).
Configuração do git com os comandos “git config --global user.email “[email]”” e “git config --global user.name “[nome]””.
Commit através do comando “git commit -m “nome do commit”
Configuração do EFS
Criação do EFS  no console AWS;
Acesso à instância ec2;
montando o EFS dentro do diretório /etc/fstab fs-07326fcfc0b337ad6.efs.us-east-1.amazonaws.com:/ home/ec2-user/efs nfs4 nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport,_netdev 0 0;
Comando sudo mount -a para montar.

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
      WORDPRESS_DB_HOST: database-2.cdhzsu6mbxkf.us-east-1.rds.amazonaws.com
      WORDPRESS_DB_USER: admin
      WORDPRESS_DB_PASSWORD: senhabanco
      WORDPRESS_DB_NAME: bancoDeDados
    volumes:
      - "/var/www/html:/home/ec2-user/efs"
volumes:
  mysql: {}

Acesso ao endereço IP da instância EC2 no navegador para verificar se o Wordpress está funcionando.

# Configuração do Load Balancer
Acesso ao serviço EC2.
Criação do Classic Load Balancer:
Configuração do LB (nome, VPC, security groups...)
Ativar as verificações de integridade de instâncias.
# Criação do target group;
Seleção das instâncias que o LB irá atuar e habilitar o balanceamento de carga.
Configuração do LB para permitir a saída tráfego de internet.
Adição de tags.
# Configuração do Auto Scaling
Criação de uma AMI da instância usada;
Criação de um lauch configuration com base na AMI criada;
Criação do grupo de Auto Scaling e associando ele ao target group.
