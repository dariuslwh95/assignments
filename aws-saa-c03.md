# exercise 2.2 - launch ec2 + ssh
- note that ec2 instances can only have their sizes changed when they are stopped
- AMIs are unique to the region
- use elastic IP addresses for persisted IPs

```
aws ec2 create-key-pair --key-name MyKeyPair --query 'KeyMaterial' --output text > MyKeyPair.pem
chmod 400 MyKeyPair.pem
aws ec2 create-security-group --group-name MySecurityGroup --description "My security group"
aws ec2 authorize-security-group-ingress --group-name MySecurityGroup --protocol tcp --port 22 --cidr 0.0.0.0/0
aws ec2 run-instances --image-id ami-06b14d295cb67ae99 --count 1 --instance-type t2.micro --key-name MyKeyPair --security-groups MySecurityGroup
aws ec2 describe-instances --query 'Reservations[*].Instances[*].PublicDnsName' --output text

ssh -i MyKeyPair.pem ec2-user@your-instance-public-dns
```

# exercise 2.4 - launch ec2, persist change, create ami from instance and launch it
- note that to connect, security group must have inbound rule permitting ipv4/ipv6 ec2 prefix e.g. ec2-18-237-4-205.us-west-2.compute.amazonaws.com
- instance can continue to run while copying
- instances must also be associated with the security groups
- note the persisted file may be in the home directory under a different user@your-instance-public-dns
```
data "aws_ami" "ubuntu" {
  most_recent = true

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*"]
  }

  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
  
resource "aws_ami_from_instance" "example" {
  name               = "my-ami"
  source_instance_id = aws_instance.web.id
  description        = "An AMI created from an existing instance with touch.txt file"

  tags = {
    Name = "MyAMI"
  }
}
```
## instance metadata
curl http://169.254.169.254/latest/meta-data/
- obtain information including ami-id, hostname, instance-id, ips, public-keys, security-groups

# exercise 2.5 create launch template with base64 encoded data
```
resource "aws_launch_template" "foo" {
  name = "myLaunchTemplate"

  image_id      = data.aws_ami.ubuntu.id
  instance_type = "t2.micro"

  user_data = base64encode(file("2.5_user_data.sh"))

}
```

# exercise 2.6 launch ec2 via aws-cli
https://docs.aws.amazon.com/cli/latest/userguide/cli-authentication-user.html
aws ec2 create-key-pair --key-name MyKeyPair --query 'KeyMaterial' --output text > MyKeyPair.pem
chmod 400 MyKeyPair.pem
aws ec2 create-security-group --group-name MySecurityGroup --description "My security group"
aws ec2 authorize-security-group-ingress --group-name MySecurityGroup --protocol tcp --port 22 --cidr 0.0.0.0/0
aws ec2 run-instances --image-id ami-06b14d295cb67ae99 --count 1 --instance-type t2.micro --key-name MyKeyPair --security-groups MySecurityGroup
aws ec2 describe-instances --query 'Reservations[*].Instances[*].PublicDnsName' --output text

ssh -i MyKeyPair.pem ec2-user@your-instance-public-dns

# exercise 2.7 clean up
aws ec2 describe-instances --query 'Reservations[*].Instances[*].InstanceId'
aws ec2 terminate-instances --instance-ids i-1234567890abcdef0
aws ec2 describe-instances --instance-ids i-1234567890abcdef0

# exercise 3.1
note requirement to make something publicly accessible
```
resource "aws_s3_bucket" "test-bucket" {
resource "aws_s3_bucket_ownership_controls" "test-bucket" {
resource "aws_s3_bucket_public_access_block" "test-bucket" {
resource "aws_s3_bucket_acl" "test-bucket" {
resource "aws_s3_object" "object" {
```

# exercise 3.2 versioning
versioning can be enforced even for static filenames using the source_hash parameter
TODO: [s3] life cycle rule configuration based on various conditions
```
resource "aws_s3_object" "object" {
  bucket = aws_s3_bucket.test-bucket.id
  key    = "test-item.sh"
  source = "test.sh"

  source_hash = filemd5("test.sh")
}

resource "aws_s3_bucket_versioning" "test-bucket" {
  bucket = aws_s3_bucket.test-bucket.id
  versioning_configuration {
    status = "Enabled"
  }
}
```

# exercise 3.3
TODO: [s3] make an object private in a public bucket
aws s3 presign s3://MyBucketName/Object --expires-in 60

# exercise 3.4 static webiste hosting
completed in cloud-resume

# exercise 4.1 - 4.2 vpc and subnet
note subnetting needs to be within the vpc's cidr

# exercise 4.3 network interface
- may result in cyclic errors
│ Error: Cycle: aws_instance.ubuntu, aws_network_interface.test_interface
- a networkinterface can only be attached to one device at a time
│ Error: attaching EC2 Network Interface (eni-0e099301489cd6bb0/i-05833e2b6ba23166b): operation error EC2: AttachNetworkInterface, https response error StatusCode: 400, RequestID: b83a4122-814c-410b-8b12-1380ce2e2fd6, api error InvalidParameterValue: Network interface 'eni-0e099301489cd6bb0' is currently in use.
- you can attach the interface in the ec2 definition or by defining a separate attachment resource
```
resource "aws_network_interface" "test_interface" {
  subnet_id   = aws_subnet.main.id
  private_ips = ["10.0.1.50"]
}

resource "aws_instance" "ubuntu" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t3.micro"
  network_interface {
    network_interface_id = aws_network_interface.test_interface.id
    device_index         = 0
  }

  tags = {
    Name = "ubuntu"
  }
}
OR 

resource "aws_network_interface_attachment" "test_attach" {
  instance_id          = aws_instance.ubuntu.id
  network_interface_id = aws_network_interface.test_interface.id
  device_index         = 1
}
```

# exercise 4.4 internet gateways and route tables
- TODO: [vpc] note the difference between the default vpc, default route table, main route table
```
resource "aws_internet_gateway" "gw" {
  vpc_id = aws_vpc.test-main.id

  tags = {
    Name = "main"
  }
}

data "aws_route_table" "main" {
  filter {
    name   = "vpc-id"
    values = [aws_vpc.test-main.id]
  }
  filter {
    name   = "association.main"
    values = ["true"]
  }
}

resource "aws_route" "default_route" {
  route_table_id         = data.aws_route_table.main.id
  destination_cidr_block = "0.0.0.0/0"
  gateway_id             = aws_internet_gateway.gw.id
}
```
# exercise 4.5 security groups
- good practices to define each rule ingress/egress as individual resources
```
resource "aws_security_group" "web-ssh" {
  name        = "web-ssh"
  description = "Allow web and SSH traffic"
  vpc_id      = aws_vpc.test-main.id

  tags = {
    Name = "allow_tls"
  }
}

resource "aws_vpc_security_group_ingress_rule" "allow_tls_ipv4" {
  security_group_id = aws_security_group.web-ssh.id
  cidr_ipv4         = "0.0.0.0/0"
  from_port         = 22
  to_port           = 22
  ip_protocol       = "tcp"
}

resource "aws_vpc_security_group_ingress_rule" "allow_web" {
  security_group_id = aws_security_group.web-ssh.id
  cidr_ipv4         = "0.0.0.0/0"
  from_port         = 80
  to_port           = 80
  ip_protocol       = "tcp"
}

resource "aws_vpc_security_group_ingress_rule" "allow_https" {
  security_group_id = aws_security_group.web-ssh.id
  cidr_ipv4         = "0.0.0.0/0"
  from_port         = 443
  to_port           = 443
  ip_protocol       = "tcp"
}
```

# exercise 4.6 NACL
- network access control list, by ingress/egress, action, protocol, cidr, ports and priority
```
resource "aws_network_acl" "test-main" {
  vpc_id = aws_vpc.test-main.id

  tags = {
    Name = "main"
  }
}

resource "aws_network_acl_rule" "foo" {
  network_acl_id = aws_network_acl.test-main.id
  rule_number    = 70
  egress         = false
  protocol       = "tcp"
  rule_action    = "allow"
  cidr_block     = "0.0.0.0/0"
  from_port      = 22
  to_port        = 22
}

resource "aws_network_acl_rule" "bar" {
  network_acl_id = aws_network_acl.test-main.id
  rule_number    = 80
  egress         = false
  protocol       = "tcp"
  rule_action    = "allow"
  cidr_block     = "42.60.92.193/32"
  from_port      = 3389
  to_port        = 3389
}
```

# exercise 4.7 elastic IP
- requires network interface/NAT gateway, route pointing to IGW to be used for inbound and outbound internet traffic
```
resource "aws_route" "default_route" {
  route_table_id         = data.aws_route_table.main.id
  destination_cidr_block = "0.0.0.0/0"
  gateway_id             = aws_internet_gateway.gw.id
}

resource "aws_eip" "one" {
  network_interface = aws_network_interface.test_interface.id
}
```

# exercise 4.8 transit gateway
- can have multiple attachments to a vpc via different subnets
```
resource "aws_ec2_transit_gateway_vpc_attachment" "gateway" {
  subnet_ids         = [aws_subnet.test-main.id]
  transit_gateway_id = aws_ec2_transit_gateway.gateway.id
  vpc_id             = aws_vpc.test-main.id
}

resource "aws_ec2_transit_gateway_vpc_attachment" "gateway_sec" {
  subnet_ids         = [aws_subnet.test-main_sec.id]
  transit_gateway_id = aws_ec2_transit_gateway.gateway.id
  vpc_id             = aws_vpc.test-main_sec.id
}
```

# exercise 4.9 black hole routes
resource "aws_ec2_transit_gateway_route" "example" {
  destination_cidr_block         = "0.0.0.0/0"
  blackhole                      = true
  transit_gateway_route_table_id = data.aws_ec2_transit_gateway_route_table.example.id
}

# exercise 5.1 RDS
resource "aws_db_instance" "default" {
  allocated_storage    = 10
  db_name              = "mydb"
  engine               = "mariadb"
  engine_version       = "10.11.8"
  instance_class       = "db.t3.micro"
  username             = "foo"
  password             = "foobarbaz"
  skip_final_snapshot  = true
}