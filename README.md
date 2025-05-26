# EC2 Instance with Python Environment

This project deploys an Amazon EC2 instance with Python 3.11 and a dedicated user environment using AWS CloudFormation.

## AWS Configuration

Before deploying the CloudFormation stack, configure your AWS credentials:

```bash
# Configure AWS CLI with your credentials
aws configure

# Enter the following information when prompted:
# AWS Access Key ID: [Your Access Key]
# AWS Secret Access Key: [Your Secret Key]
# Default region name: us-east-1
# Default output format: json

# Alternatively, you can set environment variables:
export AWS_ACCESS_KEY_ID="your_access_key"
export AWS_SECRET_ACCESS_KEY="your_secret_key"
export AWS_DEFAULT_REGION="us-east-1"
```

## Components

- **VPC**: A Virtual Private Cloud with CIDR block 10.0.0.0/16
- **Public Subnet**: A subnet with CIDR block 10.0.1.0/24
- **Internet Gateway**: Allows communication between instances in the VPC and the internet
- **Security Group**: Controls inbound and outbound traffic to the EC2 instance (SSH access)
- **EC2 Instance**: t2.micro instance running Amazon Linux 2023
- **Key Pair**: SSH key pair for secure access to the instance
- **Users**: ec2-user (default) and risksys (custom) users
- **Python Environment**: Python 3.11 with a virtual environment at /apps/risksys/venv

## Deployment

### Prerequisites

- AWS CLI installed and configured
- Permissions to create CloudFormation stacks and associated resources

### Deployment Steps

1. **Generate SSH Key Pair**:
   ```bash
   ssh-keygen -t rsa -b 2048 -f ~/.ssh/ec2_key
   chmod 400 ~/.ssh/ec2_key
   ```

2. **Deploy CloudFormation Stack**:
   ```bash
   aws cloudformation create-stack \
     --stack-name risksys-ec2-stack \
     --template-body file://create-ec2-stack.yaml \
     --parameters ParameterKey=PublicKeyMaterial,ParameterValue="$(cat ~/.ssh/ec2_key.pub)" \
     --capabilities CAPABILITY_IAM \
     --region us-east-1
   ```

3. **Monitor Stack Creation**:
   ```bash
   aws cloudformation describe-stacks \
     --stack-name risksys-ec2-stack \
     --region us-east-1
   ```

4. **Get EC2 Instance Public IP**:
   ```bash
   aws cloudformation describe-stacks \
     --stack-name risksys-ec2-stack \
     --query "Stacks[0].Outputs[?OutputKey=='PublicIP'].OutputValue" \
     --output text \
     --region us-east-1
   ```

### Accessing the Instance

Connect to the instance using SSH:

```bash
ssh -i ~/.ssh/ec2_key risksys@<PUBLIC_IP>
```

The risksys user will have:
- Python 3.11 virtual environment automatically activated
- Sudo privileges for administrative tasks
- Welcome message with usage instructions

### Testing the Environment

Once connected, verify the Python environment:

```bash
python /apps/risksys/test_python.py
```

## Cleanup

When you're done testing, remove all resources:

```bash
aws cloudformation delete-stack \
  --stack-name risksys-ec2-stack \
  --region us-east-1
```

## Features

- **Isolated Python Environment**: Dedicated virtual environment for Python development
- **Multiple Users**: Both ec2-user and risksys users available
- **Secure Access**: SSH key-based authentication
- **Automatic Setup**: Python environment automatically activated on login
- **Complete Infrastructure**: VPC, subnet, security groups, and all necessary components

## Security Considerations

- The security group allows SSH access from any IP (0.0.0.0/0). For production use, restrict this to specific IP ranges.
- Both users have sudo privileges. Consider limiting these permissions in production environments.
- The SSH key is used for authentication. Keep your private key secure.
