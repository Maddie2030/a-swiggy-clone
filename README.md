## Deploying the Swiggy clone app with Terraform, Kubernetes, and Jenkins CICD. ##

STEP 1 :


# Installation of AWS CLI ( Run these commands on bash/linux CLI ) on to your Local machine/remote machine

curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip
unzip awscliv2.zip
sudo ./aws/install
aws --version

# Installation of Terraform 

sudo apt-get update -y
sudo apt-get install -y gnupg software-properties-common curl
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | \
sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt-get update -y
sudo apt-get install -y terraform
terraform -version

STEP 2

# Configurtion of AWS CLI

---Go to your AWS account console:
 - create a new user : attach admistratoraccess policy
   after its created select the user and create an access key and download the details onto your local machine which conatins access key and secret assess key lets name it config.txt.

--Go to your local machine CLI and follow the below command

 aws configure


 ( you will be prompted to fill ACCESS KEY, SECRET ACCESS KEY, REGION, OUTPUT FORMAT details use the data from the downloaded file config.txt
 fill the first two and the other are to be left empty just click enter )

 

STEP 3


---- create a folder/directory " project_clone " and create the below files in it

# create main.tf file and add the below configuration:

 resource "aws_instance" "web" {
  ami                    = "ami-06b6e5225d1db5f46"      #change ami id for different region
  instance_type          = "t2.large"
  key_name               = "killer_test"              #change key name as per your setup
  vpc_security_group_ids = [aws_security_group.Jenkins-VM-SG.id]
  user_data              = templatefile("./install.sh", {})

  tags = {
    Name = "Jenkins-SonarQube"
  }

  root_block_device {
    volume_size = 40
  }
}

resource "aws_security_group" "Jenkins-VM-SG" {
  name        = "Jenkins-VM-SG"
  description = "Allow TLS inbound traffic"

  ingress = [
    for port in [22, 80, 443, 8080, 9000, 3000] : {
      description      = "inbound rules"
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
    Name = "Jenkins-VM-SG"
  }
}
---------------------------------------------------------------------------------------------------------
# create provider.tf file and add the below code into it:

terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

# Configure the AWS Provider
provider "aws" {
  region = "ap-south-1"     #change region as per you requirement
}
---------------------------------------------------------------------------------------------------------


STEP 4:

# Deploy an EC2 instance using terraform

Move into the folder/directory " project_clone " 
where main.tf and provider.tf files are present, in CLI

-- Run the below commands

Terraform init
Terraform plan
Terraform apply -auto-approve

which will create an instance on the AWS account

STEP 5:



