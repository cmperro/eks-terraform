# EKS Terraform

This repository contains Terraform configuration files to provision an Amazon Elastic Kubernetes Service (EKS) cluster with associated infrastructure on AWS.

## What This Creates

This Terraform configuration provisions:

- **VPC Infrastructure**: A new VPC with public and private subnets across 3 availability zones
- **EKS Cluster**: A managed Kubernetes cluster (version 1.29) with public API endpoint
- **Managed Node Group**: EC2 instances (t3.large) to run your workloads (1-3 nodes, 2 desired)
- **EBS CSI Driver**: Amazon EBS Container Storage Interface driver for persistent storage
- **IAM Roles**: Service-linked roles for EKS and EBS CSI driver functionality

## Prerequisites

Before using this repository, ensure you have:

1. **AWS CLI** installed and configured with appropriate credentials
2. **Terraform** version ~> 1.3 installed
3. **AWS IAM permissions** to create:
   - VPC, subnets, internet gateways, NAT gateways
   - EKS clusters and node groups
   - IAM roles and policies
   - EC2 instances and security groups

## Required Variables

This configuration requires three variables to be set:

| Variable | Type | Description | Example |
|----------|------|-------------|---------|
| `region` | string | AWS region where resources will be created | `us-east-2` |
| `cluster_name` | string | Name for your EKS cluster | `cmp-eks` |
| `vpc_name` | string | Name for your VPC | `cmp-vpc` |

## Quick Start

### 1. Clone and Navigate

```bash
git clone <repository-url>
cd eks-terraform
```

### 2. Configure Variables

Create a `terraform.tfvars` file (or copy from the sample):

```bash
cp terraform.tfvars.sample terraform.tfvars
```

Edit `terraform.tfvars` with your desired values:

```hcl
region = "us-east-2"
cluster_name = "my-eks-cluster"
vpc_name = "my-vpc"
```

### 3. Initialize and Apply

```bash
# Initialize Terraform
terraform init

# Review the planned changes
terraform plan

# Apply the configuration
terraform apply
```

The deployment typically takes 10-15 minutes to complete.

### 4. Configure kubectl

After successful deployment, configure kubectl to connect to your new cluster:

```bash
aws eks update-kubeconfig --region <your-region> --name <your-cluster-name>
```

For example, with the sample values:
```bash
aws eks update-kubeconfig --region us-east-2 --name my-eks
```

### 5. Verify Connection

```bash
kubectl get nodes
kubectl get pods --all-namespaces
```

## Outputs

After successful deployment, Terraform will output:

- `cluster_endpoint`: The EKS cluster API endpoint
- `cluster_name`: The name of your EKS cluster
- `cluster_security_group_id`: Security group ID for the cluster
- `region`: The AWS region where resources were created

## Network Configuration

The VPC is configured with:
- **CIDR Block**: 10.0.0.0/16
- **Private Subnets**: 10.0.1.0/24, 10.0.2.0/24, 10.0.3.0/24 (for worker nodes)
- **Public Subnets**: 10.0.4.0/24, 10.0.5.0/24, 10.0.6.0/24 (for load balancers)
- **NAT Gateway**: Single NAT gateway for cost optimization
- **Internet Gateway**: For public subnet internet access

## Node Group Configuration

The managed node group includes:
- **Instance Type**: t3.large
- **Scaling**: 1 minimum, 3 maximum, 2 desired capacity
- **AMI Type**: Amazon Linux 2
- **Placement**: Private subnets only

## Cost Considerations

This configuration is designed to be cost-effective while providing production-ready infrastructure:
- Uses a single NAT gateway (consider multiple for high availability)
- t3.large instances balance cost and performance
- EBS CSI driver included for persistent storage needs

## Cleanup

To destroy all resources:

```bash
terraform destroy
```

**Warning**: This will permanently delete your EKS cluster and all associated resources.

## Troubleshooting

### Common Issues

1. **Access Denied**: Ensure your AWS credentials have sufficient permissions
2. **Resource Limits**: Check AWS service limits for your account
3. **Region Availability**: Ensure EKS is available in your chosen region

### Getting Help

- Check AWS EKS documentation
- Review Terraform AWS provider documentation
- Examine the Terraform plan output for resource conflicts

## Security Notes

- The EKS cluster endpoint is publicly accessible (standard configuration)
- Worker nodes are in private subnets with no direct internet access
- IAM roles follow least-privilege principles
- Consider enabling additional security features like envelope encryption for production use

## Contributing

When modifying this configuration:
1. Test changes in a development environment first
2. Update this README if adding new variables or outputs
3. Follow Terraform best practices for code organization