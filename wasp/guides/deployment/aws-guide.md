# Deploying Wasp to AWS

## 1. AWS Elastic Beanstalk Setup

### Prerequisites
- AWS CLI installed
- EB CLI installed
- AWS credentials configured

### Initial Setup
```bash
# Initialize EB CLI
eb init -p node.js your-app-name

# Create .elasticbeanstalk/config.yml
cat > .elasticbeanstalk/config.yml << EOL
branch-defaults:
  main:
    environment: your-app-dev
    group_suffix: null
global:
  application_name: your-app
  branch: null
  default_ec2_keyname: null
  default_platform: Node.js 16
  default_region: us-east-1
  include_git_submodules: true
  instance_profile: null
  platform_name: null
  platform_version: null
  profile: null
  repository: null
  sc: git
  workspace_type: Application
EOL
```

### Environment Variables
```bash
# Set environment variables
eb setenv NODE_ENV=production
eb setenv DATABASE_URL=your-rds-url
eb setenv JWT_SECRET=your-secret
```

## 2. RDS Database Setup

### Create RDS Instance
```bash
# Create PostgreSQL RDS
aws rds create-db-instance \
  --db-instance-identifier your-app-db \
  --db-instance-class db.t3.micro \
  --engine postgres \
  --master-username admin \
  --master-user-password your-password \
  --allocated-storage 20

# Get connection string
aws rds describe-db-instances \
  --db-instance-identifier your-app-db \
  --query 'DBInstances[0].Endpoint.Address'
```

### Security Group Configuration
```bash
# Create security group
aws ec2 create-security-group \
  --group-name your-app-db-sg \
  --description "Database security group"

# Allow PostgreSQL access
aws ec2 authorize-security-group-ingress \
  --group-id sg-xxx \
  --protocol tcp \
  --port 5432 \
  --source-group sg-yyy
```

## 3. Deployment Configuration

### Procfile
```
web: npm start
```

### package.json
```json
{
  "scripts": {
    "start": "wasp start",
    "build": "wasp build",
    "postinstall": "wasp build"
  },
  "engines": {
    "node": "16.x"
  }
}
```

### .ebextensions Configuration
```yaml
# .ebextensions/nodecommand.config
option_settings:
  aws:elasticbeanstalk:container:nodejs:
    NodeCommand: "npm start"
    NodeVersion: 16.x
    ProxyServer: nginx
    GzipCompression: true

# .ebextensions/environment.config
option_settings:
  aws:elasticbeanstalk:application:environment:
    NODE_ENV: production
    NPM_USE_PRODUCTION: false
```

## 4. Deployment Steps

### Initial Deployment
```bash
# Create environment
eb create your-app-dev

# Deploy application
eb deploy

# Open application
eb open
```

### Continuous Deployment
```yaml
# .github/workflows/aws-deploy.yml
name: AWS Deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: einaregilsson/beanstalk-deploy@v20
        with:
          aws_access_key: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws_secret_key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          application_name: your-app
          environment_name: your-app-dev
          region: us-east-1
          version_label: ${{ github.sha }}
          deployment_package: deploy.zip
```

## 5. Scaling Configuration

### Auto Scaling
```yaml
# .ebextensions/autoscaling.config
Resources:
  AWSEBAutoScalingGroup:
    Type: AWS::AutoScaling::AutoScalingGroup
    Properties:
      MinSize: '1'
      MaxSize: '4'
      DesiredCapacity: '2'

  AWSEBAutoScalingLaunchConfiguration:
    Type: AWS::AutoScaling::LaunchConfiguration
    Properties:
      InstanceType: t3.micro
```

### Load Balancer Configuration
```yaml
# .ebextensions/loadbalancer.config
option_settings:
  aws:elb:listener:
    ListenerEnabled: 'true'
    ListenerProtocol: HTTPS
    InstancePort: '80'
    InstanceProtocol: HTTP
    SSLCertificateId: arn:aws:acm:region:id:certificate/xxx
```

## 6. Monitoring Setup

### CloudWatch Configuration
```yaml
# .ebextensions/cloudwatch.config
files:
  "/etc/cloudwatch-config.json":
    mode: "000600"
    owner: root
    group: root
    content: |
      {
        "metrics": {
          "namespace": "Your-App-Metrics",
          "metrics_collected": {
            "cpu": {
              "measurement": ["cpu_usage_idle", "cpu_usage_user", "cpu_usage_system"]
            },
            "memory": {
              "measurement": ["mem_used_percent"]
            },
            "disk": {
              "measurement": ["disk_used_percent"],
              "resources": ["/"]
            }
          }
        }
      }

commands:
  01_install_cloudwatch:
    command: yum install -y amazon-cloudwatch-agent
  02_start_cloudwatch:
    command: /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a fetch-config -m ec2 -s -c file:/etc/cloudwatch-config.json
```

### Logging
```yaml
# .ebextensions/logging.config
files:
  "/opt/elasticbeanstalk/tasks/bundlelogs.d/01-app.conf":
    content: |
      /var/log/app/*.log
    encoding: plain
```

## 7. Security Configuration

### SSL/TLS Setup
```yaml
# .ebextensions/https.config
Resources:
  sslSecurityGroupIngress:
    Type: AWS::EC2::SecurityGroupIngress
    Properties:
      GroupId: {"Fn::GetAtt" : ["AWSEBSecurityGroup", "GroupId"]}
      IpProtocol: tcp
      ToPort: 443
      FromPort: 443
      CidrIp: 0.0.0.0/0

option_settings:
  aws:elb:listener:443:
    ListenerProtocol: HTTPS
    SSLCertificateId: arn:aws:acm:region:id:certificate/xxx
    InstancePort: 80
    InstanceProtocol: HTTP
```

### WAF Configuration
```yaml
# .ebextensions/waf.config
Resources:
  WebACL:
    Type: AWS::WAFv2::WebACL
    Properties:
      Name: your-app-waf
      DefaultAction:
        Allow: {}
      Rules:
        - Name: AWSManagedRulesCommonRuleSet
          Priority: 1
          OverrideAction:
            None: {}
          Statement:
            ManagedRuleGroupStatement:
              VendorName: AWS
              Name: AWSManagedRulesCommonRuleSet
          VisibilityConfig:
            SampledRequestsEnabled: true
            CloudWatchMetricsEnabled: true
            MetricName: AWSManagedRulesCommonRuleSetMetric
```

## 8. Maintenance

### Database Backup
```bash
# Create snapshot
aws rds create-db-snapshot \
  --db-instance-identifier your-app-db \
  --db-snapshot-identifier your-app-snapshot

# List snapshots
aws rds describe-db-snapshots \
  --db-instance-identifier your-app-db
```

### Updates and Patches
```bash
# Update platform
eb upgrade

# Update dependencies
eb ssh
cd /var/app/current
npm update
```

## 9. Cost Optimization

### Resource Configuration
```yaml
# .ebextensions/cost-optimization.config
option_settings:
  aws:autoscaling:asg:
    MinSize: '1'
    MaxSize: '2'
  aws:ec2:instances:
    InstanceTypes: t3.micro,t3.small
```

### Scheduled Actions
```yaml
# .ebextensions/scheduled-actions.config
Resources:
  ScaleDownAtNight:
    Type: AWS::AutoScaling::ScheduledAction
    Properties:
      AutoScalingGroupName: {"Ref": "AWSEBAutoScalingGroup"}
      DesiredCapacity: '1'
      RecurrenceExpression: "0 0 * * *"
  
  ScaleUpMorning:
    Type: AWS::AutoScaling::ScheduledAction
    Properties:
      AutoScalingGroupName: {"Ref": "AWSEBAutoScalingGroup"}
      DesiredCapacity: '2'
      RecurrenceExpression: "0 8 * * *"
```
