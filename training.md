Terraform을 활용한 AWS VPC 및 EC2 구성 실습
======================
이번 실습은 수동으로 수행 하였던 VPC 구성과 EC2 구성 실습을 
Terraform 통해  코드형태로 구성하면서 Terraform의 동작 방식을 이해 하고 IAC 구성 방법을 익히기 위함입니다. 
이번 실습에서는 Terraform을 통해 AWS VPC 네트워크를 구성 할 수 있으며 AWS 리소스 서비스 배포를 위한 작업들을 코드 형태로 구성 및 이해 할 수 있습니다.
![VPC Architecture](img/terraform_vpc_arch.png "VPC Architecture")
# 목차

## 1. Terraform을 통한 VPC 구성

### 1.1. Terraform 모듈 initialize
main.tf
```{.bash}
provider "aws" {
    region = "ap-southeast-1"
}
```

### 1.2. EC2 인스턴스 구성
main.tf
```{.bash}
provider "aws" {
    region = "ap-southeast-1"
}

resource "aws_instance" "example" {
    ami = "ami-51a7aa2d"
    instance_type = "t2.micro"
}
```

### 1.3. EC2 설정 변경 및 변수 처리
### 1.3.1 설정 변경
main.tf
```{.bash}
provider "aws" {
    region = "ap-southeast-1"
}

resource "aws_instance" "example" {
    ami = "ami-51a7aa2d"
    instance_type = "t2.micro"
    
    tags {
        Name = "fc-[myname]-server01"
    }

}
```

#### 1.3.2 Terraform의 변수 처리
variables.tf
```{.bash}
variable "instance_tag" {
    description = "AWS instance tag Name"
}
```

main.tf
```{.bash}
provider "aws" {
    region = "ap-southeast-1"
}

resource "aws_instance" "example" {
    ami = "ami-51a7aa2d"
    instance_type = "t2.micro"
    
    tags {
        Name = "${var.instance_tag}"
    }

}
```

### 1.4. VPC 구성
#### 1.4.1 VPC 구성 설정 및 배포
main.tf
```{.bash}
provider "aws" {
    region = "ap-southeast-1"
}

resource "aws_vpc" "fc_vpc" {
    cidr_block = "10.100.0.0/16"
    tags {
          Name = "${var.tag_name}-vpc"
    }
}

resource "aws_subnet" "fc_vpc_pub_subnet_01" {
    cidr_block        = "10.100.1.0/24"
    availability_zone = "ap-southeast-1a"
    vpc_id            = "${aws_vpc.fc_vpc.id}"
    tags {
          Name = "${var.tag_name}-pub-subnet-01"
    }
}

resource "aws_subnet" "fc_vpc_pub_subnet_02" {
    cidr_block        = "10.100.2.0/24"
    availability_zone = "ap-southeast-1b"
    vpc_id            = "${aws_vpc.fc_vpc.id}"
    tags {
          Name = "${var.tag_name}-pub-subnet-02"
    }
}
```

#### 1.4.2 IGW, Route Table 정책 추가
main.tf
```{.bash}
resource "aws_internet_gateway" "fc_vpc_gw" {
  vpc_id = "${aws_vpc.fc_vpc.id}"
}

resource "aws_default_route_table" "fc_vpc_rt" {
  default_route_table_id = "${aws_vpc.fc_vpc.default_route_table_id}"

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = "${aws_internet_gateway.fc_vpc_gw.id}"
  }
}

resource "aws_route_table_association" "fc_vpc_rt_associate01" {
  subnet_id      = "${aws_subnet.fc_vpc_pub_subnet_01.id}"
  route_table_id = "${aws_vpc.fc_vpc.main_route_table_id}"
}

resource "aws_route_table_association" "fc_vpc_rt_associate02" {
  subnet_id      = "${aws_subnet.fc_vpc_pub_subnet_02.id}"
  route_table_id = "${aws_vpc.fc_vpc.main_route_table_id}"
}
```

### ETC. ssh 접속 key file 권한 변경 
```{.bash}
chmod 400 /home/ec2-user/environment/sample/key/fc_test_key.pem
```

### ETC. ubuntu 접속 방법
```{.bash}
ssh -i /home/ec2-user/environment/sample/key/fc_test_key.pem ubuntu@<ipaddr>
```