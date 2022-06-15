# C-mo-desplegar-un-cl-ster-de-Kubernetes-en-AWS-usando-Terraform

https://www.youtube.com/watch?v=Nx0ghV82g7k

https://raw.githubusercontent.com/RodrigoMvs123/C-mo-desplegar-un-cl-ster-de-Kubernetes-en-AWS-usando-Terraform/main/README.md



Amazon Elastic Kubernetes Service ( EKS ) 
Cluster AWS 

Ep3. PictureSocial - Cómo desplegar un clúster de Kubernetes en AWS usando Terraform


Infraestrutura com código
https://aws.amazon.com/pt/eks/ 
https://docs.aws.amazon.com/pt_br/eks/latest/userguide/create-cluster.html

Config File 
 
Compute ( t2.smal / lambda ) 
Network ( vpc / subnet ) 

Storage ( s3 Bucket / fsx ) 
Policies 

Terraform 
https://www.terraform.io/ ( init / plan / apply / destroy ) 

Providers 
terraform {
 required_providers {
  aws = {
    source = “hasicorp/aws”
    version = “>= 3.20.0”
    }
   }
}

Resources 
resource “aws_instance” “web” {
 ami = “ami-a1b2c3d4”
  instance_type = “t2.micro”
}

Input

variable “project_code” {
  type = string
}

variable “availability_zone” {
   type = list ( string ) 
   default = [“us-east-la”]
}
Output 

output “ec2_ip” { value = aws_instance.server.private_ip}

Locals 

locals {
   project_code = ”pso’
   environment    = “dev”
}
comon_tags = {
   project_name = “pe-${local.project.code}-${local.environment}01”
}

mkdir eks-ep3
cd eks-ep3
git clone https://github.com/aws-samples/picture-social-sample.git -b ep3
cd picture-social-sample
ls
code. 

> config tf

variable “region” {
default = “us-east-1”
description = “Region of AWS” 
}

provider “aws” {
region = var.region 
}

data “aws-availability-zones” “available” {}

locals {
cluster_name = “picturesocial-${random_interger.suffix.result}”
}

resource "random_integer" “suffix” {
min = 100
max= 999

}

> eks-cluster.tf 

module eks {
source                 = “terraform-aws-modules/eks/aws”
version              = “17.24.0”
cluster_name    = local.cluster_name
cluster_version = “1.20”
subnets             = module.vpc.privite_subnets

vpc_id = module.vpc.vpc.id

workers_group_defaults = {
root_volume_type = “gp2”
}

worker = groups = [
{
name                                        = “group-1”
instance_type                           = “t2.small”
additional_security_group_ids  = [aws_security_group.worker_group_mgmt_one 
asg_desired_capacity               = 3 
},
]

}

data “aws_eks_cluster” “cluster” {
name = module.eks.cluster_id
}

data “aws_eks_cluster_auth” “cluster” {
name = module.eks.cluster_id
}

>kubernetes.tf 

provider “kubernetes” {
host = data.aws_eks_cluster.cluster.endpoint
token = data.aws_eks_cluster_auth.cluster.token
cluster_ca_certificate = base54decode[data.aws_eks_cluster.cluster.certificate
}

> outputs.tf 

output “cluster_id” {
value = module.eks.cluster_id
}

output “cluster_endpoint” {
value = module.eks.cluster_endpoint
}

output “cluster_securtity_group_id” {
value = module.eks.cluster_security_group_id
}

output “kubeclt_config” {
value = module.eks.kubeconfig
}

output “config_map_aws_auth”  {
value = module.eks.config_map_aws_auth
}

output “region” {
value = var.region
} 

output “cluster_name” {
value = local.cluster_name
}

> security-groups.tf

resource “aws_security_group” “worker_group_mgmt_one” {
name_prefix =  “worker_group_mgmt_one”
vpc_id = module.vpc.vpc_id

ingress  {
from_port = 22 
to_port = 22
protocol = “tcp”

cibr_blocks = [
“10.0.0.0/8”,
]

}

}

resource “aws_security_group” “worker_group_mgmt_two” {
name_prefix =  “worker_group_mgmt_two”
vpc_id = module.vpc.vpc_id

ingress  {
from_port = 22 
to_port = 22
protocol = “tcp”

cibr_blocks = [
“192.168.0.0/16”,
]

}

}

resource “aws_security_group” “all_worker_mgmt” {
name_prefix =  “all_worker_management”
vpc_id = module.vpc.vpc_id

ingress  {
from_port = 22 
to_port = 22
protocol = “tcp”

cibr_blocks = [
“10.0.0.0/8”
“172.16.0.0/12”
“192.168.0.0/16”,
]

}

}


> versions.tf 
terraform {

required_providers {
aws = {
source = "hashicorp/aws"
version= “>=3.20.0”
}

random = {
source = “hashicorp/random”
version = “3.1.0”
}

local = {
source = “hashicorp/local”
version = “2.1.0”
}

null = {
source = “hashicorp/null”
version = “3.1.0”
}

kubernetes = {
source = “hashicorp/kubernetes”
version = “>= 2.0.1”
}

}

required_version = “>= 0.14”

}


> vpc.tf 

module “vpc” {
source = “terraform-aws-modules/vpc/aws”
version = “3.2.0”
}

name                              = “picturesocial-vpc”
cibr                                  = “10.0.0.0/16”
azs                                  =  data.aws.availability_zones.available.names
private_subnets              = [“10.0.1.0/24”, “10.0.2.0/24”, “10.0.3.0/24”]
public_subnets                = [“10.0.4.0/24”, “10.0.5.0/24”, “10.0.6.0/24”]
enable_nat_gateway      = true
single_nat_gateway       = true 
enable_dns_hostnames = true 

tags = {
“kubernetes.io/cluster${local.cluster.name}” = “shared”
}

public_subnet_tags = {
“kubernetes.io/cluster${local.cluster.name}” = “shared”
“kubernetes.io/role/elb”                                 = “1”
}

private_subnet_tags = {
“kubernetes.io/cluster${local.cluster.name}” = “shared”
“kubernetes.io/role/internal-elb”                    = “1” 
}
}


terraform init 
terraform plan 
terraform apply 
yes
clear
kubectl get nodes
aws eks list-clusters
terraform destroy 
yes



