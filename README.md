# UserData3
#!/bin/bash
set -euxo pipefail

# Get region via IMDSv2
TOKEN="$(curl -sS -X PUT "http://169.254.169.254/latest/api/token" \
  -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")"

AZ="$(curl -sS -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/placement/availability-zone)"

REGION="${AZ::-1}"  # chop last letter: us-east-1a -> us-east-1

# Minimal updates + deps
dnf -y update
dnf -y install ruby wget curl

# Install CodeDeploy agent from the same region
cd /tmp
wget "https://aws-codedeploy-${REGION}.s3.${REGION}.amazonaws.com/latest/install"
chmod +x ./install
./install auto

systemctl enable codedeploy-agent
systemctl start codedeploy-agent

mkdir -p /var/www/html
