# Setup Guide - DeFtunes Data Pipeline

This guide provides detailed instructions for setting up and deploying the DeFtunes data pipeline project.

## Prerequisites

Before starting, ensure you have the following installed:

- **AWS CLI** (v2.x)
- **Terraform** (v1.x)
- **Python** (3.8+)
- **dbt CLI** (v1.x)
- **Git**

## Step 1: AWS Configuration

### 1.1 Install AWS CLI
```bash
# For Windows
# Download from: https://aws.amazon.com/cli/

# For macOS
brew install awscli

# For Linux
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

### 1.2 Configure AWS Credentials
```bash
aws configure
```

You'll need to provide:
- AWS Access Key ID
- AWS Secret Access Key
- Default region (us-east-1)
- Default output format (json)

## Step 2: Project Setup

### 2.1 Clone the Repository
```bash
git clone <your-repository-url>
cd Capstone_project
```

### 2.2 Configure Sensitive Information

1. **Copy template files:**
   ```bash
   cp scripts/profiles.yml.template scripts/profiles.yml
   cp scripts/setup.sh.template scripts/setup.sh
   ```

2. **Edit profiles.yml:**
   ```bash
   nano scripts/profiles.yml
   ```
   
   Replace the placeholders with your actual values:
   - `<REDSHIFT_ENDPOINT>`: Your Redshift cluster endpoint
   - `<REDSHIFT_PASSWORD>`: Your Redshift password
   - `<REDSHIFT_USER>`: Your Redshift username

3. **Edit setup.sh:**
   ```bash
   nano scripts/setup.sh
   ```
   
   Replace the placeholders:
   - `<SOURCE_USERNAME>`: Your RDS username
   - `<SOURCE_PASSWORD>`: Your RDS password
   - `<REDSHIFT_USER>`: Your Redshift username
   - `<REDSHIFT_PASSWORD>`: Your Redshift password

## Step 3: Infrastructure Deployment

### 3.1 Initialize Terraform
```bash
cd terraform
terraform init
```

### 3.2 Review the Plan
```bash
terraform plan
```

This will show you what resources will be created. Review carefully before proceeding.

### 3.3 Deploy Infrastructure
```bash
terraform apply
```

When prompted, type `yes` to confirm the deployment.

**Note:** This will create AWS resources that may incur costs. Monitor your AWS billing.

### 3.4 Verify Deployment
Check the AWS Console to verify that the following resources were created:
- S3 buckets
- Redshift cluster
- Glue jobs
- IAM roles and policies
- VPC and networking components

## Step 4: Environment Setup

### 4.1 Run Setup Script
```bash
cd ..
source scripts/setup.sh
```

This script will:
- Set environment variables
- Copy Glue job scripts to S3
- Configure Terraform backend
- Install dbt dependencies

### 4.2 Verify Environment Variables
```bash
echo $TF_VAR_project
echo $TF_VAR_region
```

## Step 5: Data Pipeline Deployment

### 5.1 Deploy dbt Models
```bash
cd dbt_modeling
dbt deps
dbt run
```

### 5.2 Run dbt Tests
```bash
dbt test
```

### 5.3 Generate Documentation
```bash
dbt docs generate
dbt docs serve
```

## Step 6: Airflow Setup

### 6.1 Access Airflow UI
The Airflow UI should be accessible via the AWS console or the URL provided in the Terraform outputs.

### 6.2 Verify DAGs
Check that the following DAGs are available:
- `deftunes_api_pipeline`
- `deftunes_songs_pipeline`

### 6.3 Trigger Initial Run
Manually trigger the DAGs to test the pipeline:
1. Go to the Airflow UI
2. Find the DAG you want to test
3. Click "Trigger DAG"
4. Monitor the execution

## Step 7: Data Quality Monitoring

### 7.1 Check Data Quality Jobs
In the AWS Glue Console:
1. Go to Data Quality
2. Check the rule sets created
3. Review the quality assessment results

### 7.2 Monitor Pipeline Health
- Check Airflow for successful DAG runs
- Monitor Glue job logs for errors
- Verify data in Redshift tables

## Step 8: Visualization Setup

### 8.1 Access Apache Superset
The Superset instance should be accessible via the URL provided in the Terraform outputs.

### 8.2 Configure Data Sources
1. Connect to your Redshift database
2. Import the dbt models as datasets
3. Create dashboards and charts

## Troubleshooting

### Common Issues

1. **Terraform Apply Fails**
   - Check AWS credentials
   - Verify region settings
   - Ensure sufficient permissions

2. **dbt Connection Errors**
   - Verify Redshift endpoint
   - Check credentials in profiles.yml
   - Ensure Redshift cluster is running

3. **Glue Job Failures**
   - Check S3 bucket permissions
   - Verify IAM roles
   - Review CloudWatch logs

4. **Airflow DAG Failures**
   - Check Glue job status
   - Verify S3 paths
   - Review Airflow logs

### Debugging Commands

```bash
# Check AWS credentials
aws sts get-caller-identity

# Check Terraform state
terraform show

# Check dbt connection
dbt debug

# List S3 buckets
aws s3 ls

# Check Redshift connection
psql -h <redshift-endpoint> -U <username> -d dev
```

## Cost Optimization

### AWS Cost Management
- Monitor AWS billing dashboard
- Set up billing alerts
- Use AWS Cost Explorer to track spending
- Consider using AWS Budgets

### Resource Cleanup
To avoid ongoing costs, destroy the infrastructure when not needed:
```bash
cd terraform
terraform destroy
```

**Warning:** This will delete all created resources and data.

## Security Considerations

### Best Practices
- Use IAM roles instead of access keys when possible
- Enable CloudTrail for audit logging
- Use VPC endpoints for private communication
- Regularly rotate credentials
- Enable encryption at rest and in transit

### Network Security
- Use private subnets for databases
- Configure security groups properly
- Enable VPC flow logs for monitoring

## Monitoring and Alerting

### CloudWatch Alarms
Set up alarms for:
- Glue job failures
- Redshift cluster health
- S3 bucket access
- API Gateway errors

### Logging
- Enable CloudWatch logs for all services
- Set up log retention policies
- Monitor error rates and performance

## Next Steps

After successful deployment:

1. **Data Validation**
   - Verify data quality metrics
   - Check data completeness
   - Validate business logic

2. **Performance Optimization**
   - Monitor query performance
   - Optimize Redshift cluster size
   - Tune Glue job parameters

3. **Scaling**
   - Plan for data volume growth
   - Consider multi-region deployment
   - Implement backup strategies

4. **Documentation**
   - Document data lineage
   - Create runbooks for operations
   - Maintain architecture diagrams

---

For additional support, refer to the main README.md file or create an issue in the repository. 