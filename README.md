# Terraform-Practice-Project

<img src="https://miro.medium.com/max/1400/1*7x7SmXVPuUZyP9GbOISZbA.png">

<h3 Automate your AWS Cloud Architecture with Terraform />

This is the IAC practice to create the vpc single layered Architechture in AWS Cloud
- Terraform
- Virtual Private Cloud(vpc)
- EC2
- VS Code

All components are Terraform based

### With Terraform

#### To start the application The main terraform commands are
```
1) Terraform init
2) Terraform Plan
3) Terraform apply
4) Terraform destroy
```

Step 1: Create docker network

```
    resource "aws_vpc" "prod-vpc" {
  cidr_block = "10.0.0.0/16"
  tags = {
    Name = "main production"
  }
}
```

Step 2: create internet gateway
```
    resource "aws_internet_gateway" "gw" {
  vpc_id = aws_vpc.prod-vpc.id

  tags = {
    Name = "main production gateway"
  }
}
```
Step 3: Create Custom Route Table
```
resource "aws_route_table" "prod-route-table" {
  vpc_id = aws_vpc.prod-vpc.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.gw.id
  }
   route {
    ipv6_cidr_block        = "::/0"
    gateway_id = aws_internet_gateway.gw.id
  }

  tags = {
    Name = "prod-table"
  }
}
```
Step 4:create subnet
```
resource "aws_subnet" "prod-subnet" {
  vpc_id     = aws_vpc.prod-vpc.id
  cidr_block = "10.0.1.0/24"
  availability_zone = "us-east-1a"

  tags = {
    Name = "prod-subnet"
  }
}
```
Step 5:  Associate subnet with route table
```

resource "aws_route_table_association" "a" {
  subnet_id      = aws_subnet.prod-subnet.id
  route_table_id = aws_route_table.prod-route-table.id
}

```

Step 6: Create securitu group to port 22,80,443
```
resource "aws_security_group" "allow_web" {
  name        = "allow_web_traffic"
  description = "Allow web traffic inbound traffic"
  vpc_id      = aws_vpc.prod-vpc.id

  ingress {
    description      = "HTTPS"
    from_port        = 443
    to_port          = 443
    protocol         = "tcp"
    cidr_blocks      = ["0.0.0.0/0"]
  }
  ingress {
    description      = "HTTPS"
    from_port        = 80
    to_port          = 80
    protocol         = "tcp"
    cidr_blocks      = ["0.0.0.0/0"]
  }
  ingress {
    description      = "HTTPS"
    from_port        = 2
    to_port          = 2
    protocol         = "tcp"
    cidr_blocks      = ["0.0.0.0/0"]
  }

  egress {
    from_port        = 0
    to_port          = 0
    protocol         = "-1"
    cidr_blocks      = ["0.0.0.0/0"]
    ipv6_cidr_blocks = ["::/0"]
  }

  tags = {
    Name = "allow_web"
  }
}

```
Step:7  Creating Network Interface
```
resource "aws_network_interface" "web_server_nic" {
  subnet_id       = aws_subnet.prod-subnet.id
  private_ips     = ["10.0.1.50"]
  security_groups = [aws_security_group.allow_web.id]
}
```

Step 8: Assign an elastic ip
```
resource "aws_eip" "one" {
  vpc                       = true
  network_interface         = aws_network_interface.web_server_nic.id
  associate_with_private_ip = "10.0.1.50"
  depends_on = [aws_internet_gateway.gw]
}

``` 
Step 9: creating ubuntu server
```
resource "aws_instance" "web-server-instance" {
  ami = "ami-08c40ec9ead489470"
  instance_type = "t2.micro"
  availability_zone = "us-east-1a"
  key_name = "main-key"

  network_interface {
    device_index = 0
    network_interface_id = aws_network_interface.web_server_nic.id

  }


  user_data = <<-EOF
      #/bin/bash
      sudo apt update -y
      sudo apt install apache2 -y
      sudo systemctl start apache2
      sudo bash -c 'echo your very first web server > vra/www/html/index.html'
      EOF

  tags ={
    Name = "web-server"
  }

}
```
After Completing the code Execute the following Commands
```
1) Terraform init
2) Terraform plan
3) Terraform apply
```
After Completing all the commands set up your <b>AWS</b> Console
Check out The web server is created and it is running in instance 
After the completion of Coud Architechture
```
Terraform destroy
```
After Automate the Infrastructure destroy all the resources in your Aws account 
refer the Terraform documentation
https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/vpc
refer kavi community in youtube for clear cut idea
