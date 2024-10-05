---
# Terraform S3 and DynamoDB Remote State Example

This project demonstrates how to set up AWS S3 and DynamoDB for storing Terraform's remote state file and implementing state locking to ensure safe and reliable infrastructure provisioning.

## Prerequisites

- Terraform installed on your local machine.
- AWS account with proper IAM permissions (S3, DynamoDB, and EC2).
- AWS CLI configured with access credentials.
- Ensure that the correct region is set up in your environment (in this case, `ap-south-1`).

## Project Structure

- `DynamodbS3/main.tf`: Creates the S3 bucket and DynamoDB table required for remote state storage and state locking.
- `demo/backend.tf`: Configures Terraform to use S3 for storing the remote state and DynamoDB for locking.
- `demo/main.tf`: Creates an AWS EC2 instance using Terraform.

### File Descriptions

#### DynamodbS3/main.tf

This file sets up the following resources:

- **S3 Bucket**: This bucket stores the Terraform state file.
- **DynamoDB Table**: This table manages the state lock to ensure that only one Terraform process modifies the state file at any given time.

```hcl
provider "aws" {
  region = "ap-south-1"
}

# S3 bucket for storing remote state
resource "aws_s3_bucket" "example" {
  bucket = "akash-dynamodb-statelock-demo-s3"
}

# DynamoDB table for state locking
resource "aws_dynamodb_table" "dynamodb-terraform-state-lock" {
  name           = "terraform-state-lock-dynamo"
  hash_key       = "LockID"
  read_capacity  = 20
  write_capacity = 20

  attribute {
    name = "LockID"
    type = "S"
  }

  tags = {
    Name = "DynamoDB Terraform State Lock Table"
  }
}
```

#### demo/backend.tf

This file configures Terraform to use the S3 bucket and DynamoDB table created earlier.

```hcl
terraform {
  backend "s3" {
    bucket         = "akash-dynamodb-statelock-demo-s3"
    key            = "default/terraform.tfstate" # Location of the state file
    region         = "ap-south-1"
    dynamodb_table = "terraform-state-lock-dynamo" # For state locking
    encrypt        = "true"
  }
}
```

#### demo/main.tf

This file creates an EC2 instance in the `ap-south-1` region using the `t2.micro` instance type and the specified Amazon Machine Image (AMI).

```hcl
provider "aws" {
  region = "ap-south-1"
}

# EC2 instance
resource "aws_instance" "example" {
  ami           = "ami-0b99c7725b9484f9e"
  instance_type = "t2.micro"
}
```

## Usage

### Step 1: Initialize the Terraform Backend

1. Navigate to the `demo/` directory:
   ```bash
   cd demo
   ```

2. Initialize Terraform to set up the backend:
   ```bash
   terraform init
   ```

### Step 2: Apply the Configuration

1. Review the infrastructure plan:
   ```bash
   terraform plan
   ```

2. Apply the changes to create the EC2 instance:
   ```bash
   terraform apply
   ```

   This will create the necessary resources on AWS (S3, DynamoDB, and EC2).

### Step 3: Cleanup

To destroy the resources created by Terraform, run:

```bash
terraform destroy
```

## Additional Notes

- **Remote State**: Using an S3 bucket to store the Terraform state ensures that multiple team members can work together on the same infrastructure.
- **State Locking**: The DynamoDB table prevents multiple Terraform executions from modifying the state at the same time, providing safe concurrency handling.
- **Security**: The `encrypt` option is set to `true` in the `backend.tf` file to ensure that the state file is encrypted at rest.

## Resources

- [Terraform Documentation](https://www.terraform.io/docs)
- [AWS S3](https://aws.amazon.com/s3/)
- [AWS DynamoDB](https://aws.amazon.com/dynamodb/)

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---
