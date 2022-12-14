Titel: Webserver in AWS mit CLI erstellen - Dokumentation
Author: Stefan Sell
Date: 07-11-2022
Keywords: AWS, CLI, Apache, Webserver

# Webserver in AWS mit CLI erstellen - Dokumentation

## Die aktuelle Version des CLI ermitteln
    aws --version   
## Die aktuelle Konfiguration anzeigen
    aws configure list     
## EC2 Instance
### AMI (Amazon Machine Image) anzeigen
    aws ec2 describe-images --owners amazon --filters "Name=name, Values=amzn2-ami-hvm-2.0.????????.?-x86_64-gp2" "Name=state, Values=available" --query "reverse(sort_by(Images, &Name))[:1].ImageId" --output text
die ami.id kopieren (ami-070b208e993b59cea9)
### Instance Type
in der AWS Konsole unter Instance/Instance Types t2.micro auswählen
![InstanceType](https://miro.medium.com/max/720/1*ut8SVOjduLuQCxZCx5UhFA.png)
## Key Pair erstellen und anzeigen
    aws ec2 create-key-pair --key-name StefanS-Linux-Key --query 'KeyMaterial' --output text > StefanS-Linux-Key.pem
    aws ec2 describe-key-pairs --key-name StefanS-Linux-Key
## Security Group erstellen und Regeln hinzufügen
Security Group ID: sg-06c3d2869b88ec830

Port 22 für SSH und Port 80 für HTTP werde für eingehende Verbindungen freigegeben

    aws ec2 create-security-group --group-name StefanS-MyWebSG --description "Allows SSH and HTTP connections for the Web Server"
    aws ec2 authorize-security-group-ingress --group-id sg-06c3d2869b88ec830 --protocol tcp --port 22 --cidr 0.0.0.0/0
    aws ec2 describe-security-groups --group-ids sg-06c3d2869b88ec830
## EC2 Script zum Start des Apache Webserver erstellen
Datei webscript.sh in C:\Benutzer\Stefan.Sell erstellen

folgende Zeilen hinzufügen und die Datei speichern:

    #!/bin/bash
    yum update -y
    yum install httpd -y
    systemctl start httpd
    systemctl enable httpd
    
## EC2 Instance erstellen
    aws ec2 run-instances --image-id (ami-070b208e993b59cea9 --count 1 --instance-type t2.micro --key-name StefanS-Linux-Key --security-group-ids sg-06c3d2869b88ec830 --user-data file://webscript.sh
## EC2 Instance und Webserver überprüfen
### Status überprüfen
    aws ec2 describe-instances
### als ec2-user über ssh einloggen
    ssh -i StefanS-Linux-Key ec2-user@18.185.118.99
### HTTP-Netzwerkverbindungen überprüfen
    sudo systemctl status httpd
### PublicIPaddress kopieren und im Browser einfügen
Public IP Address: 18.185.118.99

![Apache](https://miro.medium.com/max/720/1*dQz8HxEfH0nzgI8Tz2eA3w.png)

## Referenz
[How to use AWS CLI to launch an EC2 Web Server with Apache](https://towardsaws.com/how-to-use-aws-cli-to-launch-an-ec2-web-server-with-apache-9c20d07e07be) 
