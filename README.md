Today we are going to deploy reddit app in aws ec2 using terraform as a infrastructure building tool and then we automate the whole process using the jenkins which allows us continuous integration and continuous deployment (CI/CD) and then we setup Promotheus and grafana for monitering

Completion steps →
Step 1 → Setup Terraform and configure aws on your local machine

Step 2 → Building a simple Infrastructure from code using terraform

Step 3 → Setup Sonarqube and jenkins

Step 4 → CI/CD pipeline

Step5 →Monitering via Prmotheus and grafana

Step 6 → Terraform Destroy

Basic Setup→
open your terminal and make a separate folder for reddit app →mkdir Reddit
cd Reddit
clone the github repo
git clone https://github.com/Tharun-P7/Reddit-clone-deployment.git
Step 1 → Setup Terraform and configure aws on your local machine
1. Setup Terraform
To install terraform copy and paste the below commands

sudo su
snap install terraform --classic
which terraform

2. Configure aws
create an IAM user
go to your aws account and type IAM

click on IAM

3. click on user →create user

I have already an amazon user if you want you create a new user checkout the below steps


4. Give a name to your user and tick on provide user access to management console and then click on I want an IAM user option


5. choose a password for your user →click next

6. Attach the policies directly to your iam user → click next

Note →I will provide the administrator accesss for now but we careful while attaching the policies on your workapce



review and create user

7. click on create user


8. download your password file if it is autogenerated otherwise it is your’s choice


9. Now click on your IAM user →security credentials


10. scroll down to access keys and create an access keys


11.choose aws cli from the options listed


12. click next and download you csv file for username and password


13. go to your terminal and type →aws configure

14. Now it is ask to your access key and secret key for this open your csv file and paste the access and secret key and remain everything default


15. Now you are ready to configure aws from your terminal

Step 2 → Building a simple Infrastructure from code using terraform
go to folder → cd EC2-terraform
there are three files present main.tf, EC2.sh , provider.tf
open the file →vim Main.tf

4. change this section → ami = # your ami id , key_name= #your key pair if any

Now run terraform commands →
main.tf includes userdata which links EC2.sh file on which execution install jenkins,docker,trivy,and start the sonarqube container on port 9000
resource "aws_security_group" "Jenkins-sg" {
  name        = "Jenkins-Security Group"
  description = "Open 22,443,80,8080,9000"
# Define a single ingress rule to allow traffic on all specified ports
  ingress = [
    for port in [22, 80, 443, 8080, 9000, 3000] : {
      description      = "TLS from VPC"
      from_port        = port
      to_port          = port
      protocol         = "tcp"
      cidr_blocks      = ["0.0.0.0/0"]
      ipv6_cidr_blocks = []
      prefix_list_ids  = []
      security_groups  = []
      self             = false
    }
  ]
egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
tags = {
    Name = "Jenkins-sg"
  }
}
resource "aws_instance" "web" {
  ami                    = "ami-0c7217cdde317cfec"#change the ami id
  instance_type          = "t2.large"
  key_name               = "my key"
  vpc_security_group_ids = [aws_security_group.Jenkins-sg.id]
  user_data              = templatefile("./install_jenkins.sh", {})
tags = {
    Name = "Reddit clone"
  }
  root_block_device {
    volume_size = 30
  }
}
terraform init
terraform validate
terraform plan
terraform apply --auto-approve


terraform init,validate and plan output


terraform apply

Go to your aws console and checkout the ec2 instances



ports that are listed in main.tf to be opened

Here we see Reddit app instance is created by terraform with the given configuration