# Terraform
Un pequeno ejemplo de como utilizar terraform

### Pre-requisitos 📋
1. Tener instalado Terraform
Primero instalamos el manejador de paquetes snap
```
sudo apt-get install snap
```
2. Instalamos Terraform
```
sudo snap install terraform
```
**unicos requisitos**

## Comenzando 🚀
1. Para empezar tenemos que crear nuestro archivo con extension .tf
```
nano nombre.tf
```
2. Una vez dentro del editor, copiamos el siguiente codigo, que es una configuracion basica para una instancia en Amazon Web Service
```
#variable "access_key" {}
#variable "secret_key" {}


provider "aws" {
  access_key = "<ACCESS-KEY>"
  secret_key = "<SECRET-KEY>"
  region     = "us-east-2"
}

#Lo comente para que no vuelva a crear el mismo recurso
#resource "aws_key_pair" "seminario_Junio2019" {
#  key_name   = "seminario_Junio2019"
# public_key = "<PUBLIC-KEY> root@docker-seminario1"
#}

resource "aws_security_group" "<NOMBRE-RECURSO-SEGURIDAD>" {
  name        = "rancher_fw_rules3"
  description = "Rancher firewall rules"
#  vpc_id      = "${aws_vpc.main.id}"
#  vpc_id     = "${module.vpc.vpc_id}"

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  ingress {
    from_port   = 2376
    to_port     = 2376
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  ingress {
    from_port   = 2379
    to_port     = 2380
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  ingress {
    from_port   = 3306
    to_port     = 3306
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  ingress {
    from_port   = 6379
    to_port     = 6379
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  ingress {
    from_port   = 6443
    to_port     = 6443
    protocol    = "udp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  ingress {
    from_port   = 8472
    to_port     = 8472
    protocol    = "udp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  ingress {
    from_port   = 9099
    to_port     = 9099
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  ingress {
    from_port   = 10250
    to_port     = 10250
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  ingress {
    from_port   = 10254
    to_port     = 10254
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  ingress {
    from_port   = 30000
    to_port     = 32767
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  ingress {
    from_port   = 30000
    to_port     = 32767
    protocol    = "udp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port       = 0
    to_port         = 65535
    protocol        = "tcp"
    cidr_blocks     = ["0.0.0.0/0"]
#    prefix_list_ids = ["pl-12c4e678"]
  }
  egress {
    from_port       = 0
    to_port         = 65535
    protocol        = "udp"
    cidr_blocks     = ["0.0.0.0/0"]
#    prefix_list_ids = ["pl-12c4e678"]
  }


  tags {
    Name = "rancher_fw_rules"
  }

}



resource "aws_instance" "ubuntu-terraform" {
#east-2
  ami           = "ami-0c55b159cbfafe1f0" 
  instance_type = "t2.micro"
  key_name = "<NOMBRE-DEL-ARCHIV.PEM>"
  root_block_device = {
    volume_type           = "gp2"
    volume_size           = 50
    delete_on_termination = true
  }

  provisioner "remote-exec" {
    inline = [
      "export PATH=$PATH:/usr/bin:/bin/sbin",
      "sudo echo 'Waiting for the network'",
      "sudo apt-get update",
      "sudo echo 'Instalacion de Docker'",
      "sudo apt-get -y install docker.io",
      "sudo echo 'Clonacion de images de DockerHub.com'",
      "sudo docker pull andreeavalos/database:last ",
      "sudo docker pull andreeavalos/server:last ",
      "sudo docker pull andreeavalos/api:last ",
      "sudo echo 'cambio de nombres'",
      "sudo docker tag andreeavalos/database:last database:latest",
      "sudo docker tag andreeavalos/server:last server:latest",
      "sudo docker tag andreeavalos/api:last api:latest",
      "sudo echo 'Eliminacion de tags innecesarios'",
      "sudo docker rmi andreeavalos/database:last ",
      "sudo docker rmi andreeavalos/server:last ",
      "sudo docker rmi andreeavalos/api:last ",
      "sudo echo ''",
      "sudo docker run -it --name api-node api",
      "exit",
      "sudo docker run -it --name server-node -p 80:80 server",
      "exit",
      #"sudo docker run -it --name database database",
      #"exit",
    ]

    connection {
      type     = "ssh"
      private_key = "${file("id_rsa")}"
      user     = "ubuntu"
      timeout  = "5m"
    }
  }

  depends_on = ["aws_security_group.<NOMBRE-RECURSO-SEGURIDAD>"]

  vpc_security_group_ids = ["${aws_security_group.<NOMBRE-RECURSO-SEGURIDAD>.id}"]

}

output "ubuntu_ip" {
#  value = "${aws_eip.ip2.public_ip}"
  value = "${aws_instance.ubuntu-terraform.public_ip}"
}


```
3. Para obtener la public key
```
ssh-keygen -y -f archivo.pem
```
Les mostrara algo asi
```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCuVMOvlxnFbm1zvyrIU7Dl9SGjroCggVdYMKGCCpFtTTvSbwTYXJCzDpkvKTpY1L+KKyXHx4rumZZSnPBwTNBI7AN708909weBs/3o1U8YSGw0d8z25fndCmGy/4lIKzZHdSVwURt/2Qi0BggU6A0W6PvWWKMHGOJLMsmgGqV05DYkZGZtl0pVfQgjGHMBqZKNeJzTylBgJOEmnnO6iiaJfq/vfuZj1bap0rG2zAUwZ7nwKnTnOBEjt+atdnG0HUHLG9A+ayyrAY0GsBGerpOglyQMli3p0QTews3uaPlzLQrMNYGyNC9hIjeGbQMdVYoVFtrGaKCDAIYTnE5qP+uh+vL7jOT
```
4. Copiamos el archivo .pem y lo renombramos como id_rsa

## Ejecutando las pruebas ⚙️
1. Iniciamos el ambito de terraform en la carpeta donde creamos nuestro archivo .tf
```
terraform init
```
2. Aplicamos el plan
```
terraform plan
```
3. Lo ejecutamos para que cree la instancia
```
terraform apply
```

## Deployment 📦
* [AWS](https://docs.aws.amazon.com)- Documentacion
