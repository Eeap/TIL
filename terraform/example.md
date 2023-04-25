# Terraform Example

테라폼 코드는 확장자가 .tf인 HCL로 작성되고 HCL은 선언적 언어이므로 원하는 인프라를 설명하기 위해서 코드를 작성해야한다.
코드의 형식은 다음과 같다.
``` HCL
resource "<PROVIDER>_<TYPE>" "<NAME>" {
    [CONFIG..]
}
```

예를 들어 ec2 인스턴스를 배포하려고 한다면 다음과 같은 코드를 작성하면 된다.
```HCL
resource "aws_instance" "example" {
  ami           = "ami-830c94e3"
  instance_type = "t2.micro"
}
```
여기서 ami는 ec2인스턴스를 생성하는 아마존 머신 이미지의 약자이고 aws 마켓플레이스에서 무료 및 유료 ami를 찾거나 직접 ami를 생성할 수 있다.
instance_type은 실행할 instance의 유형이고 ec2의 인스턴스 타입은 리소스에 따라 여러 유형이 존재한다.

main.tf에 위와 같은 코드를 작성한 다음 terraform init을 실행해서 테라폼에 코드를 스캔하도록 지시하고 어느 공급자인지 확인하고 필요한 코드를 다운받도록 해야한다. 기본적으로 테라폼 코드는 테라폼의 .terraform 폴더에 다운로드된다.

terraform plan 명령어를 사용하면 실제로 변경되기 전에 테라폼이 수행할 작업을 확인할 수 있고 실제 운영 환경에서 적영하기 전에 코드의 온전성을 검사할 수 있는 좋은 방법이다. 기본적으로 + 표시는 추가, - 표시는 삭제, ~ 표시는 수정을 의미한다.

그 다음 terraform apply를 입력하면 plan과 똑같은 결과값이 나오고 이 plan을 진행할 것인지 확인하라는 메시지를 출력한다. yes를 입력하면 ec2하나가 배포되는 것을 콘솔을 통해 확인할 수 있다.

ec2 instance의 config에는 위에 옵션 말고 tag나 다른 것도 추가가 가능하다.
```HCL
resource "aws_instance" "example" {
  ami = "ami-830c94e3"
  instance_type = "t2.micro"

  tags = {
    Name = "terraform-example"
  }
}
```
위와 같은 내용으로 다시 terraform apply 명령어를 실행하면 테라폼은 구성 파일을 위해 생성된 모든 리소스를 추적하므로 ec2 인스턴스가 이미 존재한다는 것을 알고 있기 때문에 refreshing state라는 메시지가 뜨게 되고 실제 콘솔 창도 들어가면 Name값만 변경된 것을 확인할 수 있다.

*.tfstate파일은 테라폼이 상태를 저장하는데 사용하고 여기에는 리소스에 대한 정보도 포함하고 있기 때문에(유저 정보도 포함) gitignore에 반드시 추가해줘야한다.

user_data를 설정하면 스크립트를 실행할 수도 있다. 여기서 <<-EOF 및 EOF는 heredoc 구문을 이용해 줄 바꿈 문자를 삽입하지 않고도 여러 줄로 된 코드를 작성할 수 있다. 그리고 웹 서버에 접근하기 위해선 해당 포트에 대한 인바운드 규칙을 수정해서 트래픽을 받아들여야하는데 아래처럼 security group을 생성해서 설정할 수 있다. CIDR 블록을 0.0.0.0/0으로 설정하면 8080포트로 오는 모든 요청에 대해 수용함을 의미한다. 또한, 보안 그룹을 ec2에 설정을 해야하고 참조 표현식을 이용해서 종속성을 작성할 수 있고 이러한 종속성 구문을 테라폼이 알아서 분석해서 종속성 그패를 작성하여 리소스 생성 순서를 자동으로 결정한다.(표현식 -> <PROVIDER>_<TYPE>.<NAME>.<ATTRIBUTE>). 종속성 그래프는 `$ terraform graph`라는 명령어를 통해 볼 수 있다.
```HCL
resource "aws_instance" "example" {
  ami = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  vpc_security_group_ids = [aws_security_group.example_sg.id]

  user_data = <<-EOF
                #!/bin/bash
                echo "Hello, World!" > index.html
                nohup busybox httpd -f -p 8080 &
                EOF
  tags = {
    Name = "terraform-example"
  }
}

resource "aws_security_group" "example_sg" {
    name = "terraform-example-instance"

    ingress {
      cidr_blocks = [ "0.0.0.0/0" ]
      from_port = 8080
      protocol = "tcp"
      to_port = 8080
    } 
}
```

테라폼은 입력 변수를 정의하게 해서 코드가 중복되지 않도록 구성을 관리하기가 쉬운 편이다. 변수 선언 구문은 아래와 같고 변수로는 description, default, type이 있다.
description은 말그대로 설명을 뜻하고 default 같은 경우는 값을 지정하는데 사용된다. type은 variable의 type을 선언할 수 있고 string, number, bool, list, set, object, map, tuple등이 있다. 유형을 지정하지 않으면 any로 인식하고 default 같은 경우도 지정하지 않으면 대화 형식으로 사용자에게 변수를 묻는다. default 말고도 값을 설정하는 방법으로는 명령어를 날릴때 -var 옵션을 주면 된다.
```HCL
variable "example" {
  description = "example variable"
  type = number
  default = 8080
}
아래 처럼 조건을 결합해서 사용할 수도 있다.
variable "example" {
  description = "example variable"
  type = list(number)
  default = [8080]
}
variable "example" {
  description = "example variable"
  type = map(number)
  default = {
    key1=1
    key2=2
  }
}
variable "example" {
  description = "example variable"
  type = object({
    name = string
    age = number
    enabled = bool
  })
  default = {
    name = "test"
    age = 20
    enabled = true
  }
}
만약 default를 정의하지 않았을 경우엔
terraform plan -var "example=8080"
환경 변수를 통해 변수 설정할 경우엔(TF_VAR_<name>)
export TF_VAR_server_port=8080
terraform plan
```

위에서 만들어놓은 입력 변수를 이용해서 이제 매개 변수 값을 설정할 수 있다.
```HCL
resource "aws_security_group" "example_sg" {
    name = "terraform-example-instance"


    ingress {
      cidr_blocks = [ "0.0.0.0/0" ]
      from_port = var.server_post
      protocol = "tcp"
      to_port = var.server_post
    } 
  
}
```

테라폼은 입력 변수 뿐만 아니라 출력 변수 설정도 가능하다. 변수는 description과 sensitive, value가 있고 value는 반드시 필요하고 앞에 두개는 옵션이다. sensitive는 이제 보안에 예민한 값의 경우 true로 설정하면 출력을 기록하지 않도록 한다. 결과값은 terraform output 명령어를 사용하여 확인하거나 apply한 후 결과값으로 확인할 수 있다.

오토스케일링 그룹을 이용하면 ec2 인스턴스의 클러스터 시작, 상태 모니터링 등의 작업들을 자동으로 처리할 수 있다. 먼저 해야할 것은 ec2 인스턴스를 어떻게 구성할 것인지 설정하는 시작 구성을 만들어야한다.
```HCL
resource "aws_autoscaling_group" "example" {
  launch_configuration = aws_launch_configuration.example.name
  vpc_zone_identifier = data.aws_subnet_ids.default.ids

  target_group_arns = [aws_lb_target_group.alb_tg.arn]
  health_check_type = "ELB"
  min_size = 2
  max_size = 10

  tag {
    key = "Name"
    value = "terraform-asg-example"
    propagate_at_launch = true
  }
}

resource "aws_launch_configuration" "example" {
  image_id = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"

  security_groups = [aws_security_group.example_sg.id]

  user_data = <<-EOF
              #!/bin/bash
              echo "Hello, World!" > index.html
              nohup busybox httpd -f -p ${var.server_port} &
              EOF
  lifecycle {
    create_before_destroy = true
  }
}
```
위에서 vpc 변수는 인스턴스를 어느 vpc에 배포할지 결정하는 변수이고 lifecycle은 테라폼은 보통 리소스를 교체할 때 이전 리소스를 먼저 삭제한 다음 대체 리소스를 생성하기 때문에 이전 리소스에 대한 참조가 있어서 삭제할 수 없다. 그래서 먼저 만든 다음 기존 인스턴스를 삭제하도록 하기 위해서 create_before_destroy라는 변수를 이용한다. 여기서 vpc의 해당 서브넷들의 id 값을 가져오기 위해선 데이터 소스라는 것을 사용해야하는데 이것은 테라폼이 공급자에서 가져온 읽기 전용 정보이다.

```HCL
기본 vpc의 데이터를 조회하는 방법
data "aws_vpc" "default" {
  default = true

}
아래는 권장하는 방법으로 subnet_ids를 가져오는 방법
data "aws_subnets" "default" {
  filter {
    name = "vpc-id"
    values = [data.aws_vpc.default.id]
  }
}

# 아래는 terraform에서 deprecate된다고 해서 추후에는 subnets 쓸것을 권장(https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/subnet_ids)
# data "aws_subnet_ids" "default" {
#   vpc_id = data.aws_vpc.default.id
# }
```

ASG는 각각 고유한 ip 주소를 가진 여러 개의 서버를 배포하기 때문에 이를 하나의 ip주소로 제공할 필요가 있다. 이를 위해선 로드 밸런서를 이용할 필요가 있고 aws에서 ELB 서비스를 사용하면 된다.
ELB 종류로는 ALB, NLB, CLB가 있고 대부분은 ALB나 NLB를 사용하고 ALB는 http, https의 트래픽 처리에 적합하고 NLB는 tcp, udp 및 tls 트래픽 처리에 적합하다. ALB는 listener, listener rule, target groups로 구성되어 있다.
```HCL
resource "aws_lb" "example" {
  name = "terraform-lb-example"
  load_balancer_type = "application"
  subnets = data.aws_subnets.default.ids
  security_groups = [aws_security_group.lb_example_sg.id]
}
리스너는 기본 http 포트인 80번 포트를 수신하고, 리스너 규칙과 일치하지 않는 요청에 대해 기본 응답으로 404페이지를 보내도록 구성
resource "aws_lb_listener" "http" {
  load_balancer_arn = aws_lb.example.arn
  port = 80
  protocol = "HTTP"
  
  default_action {
    type = "fixed-response"
    
    fixed_response {
      content_type = "text/plain"
      message_body = "404: page not found"
      status_code = 404
    }
  }
}
추가적으로 alb 보안 그룹에 대해 inbound는 80번 포트를 허용하고 아웃바운드로는 health check가 가능하도록 수정
resource "aws_security_group" "lb_example_sg" {
  name = "terraform-example-alb"

  ingress {
    cidr_blocks = [ "0.0.0.0/0" ]
    from_port = 80
    protocol = "tcp"
    to_port = 80
  }
  egress {
    cidr_blocks = [ "0.0.0.0/0" ]
    from_port = 0
    protocol = "-1"
    to_port = 0
  }
}
ASG의 대상 그룹을 생성해서 HTTP로 상태를 점검하고 matcher와 동일한 응답을 받아야만 정상으로 간주. autoscaling group 리소스에 target group을 아래 대상 그룹으로 지정.
resource "aws_lb_target_group" "alb_tg" {
  name = "terraform-alb-tg"
  port = var.server_port
  protocol = "HTTP"
  vpc_id = data.aws_vpc.default.id

  health_check {
    path = "/"
    protocol = "HTTP"
    matcher = "200"
    interval = 15
    timeout = 3
    healthy_threshold = 2
    unhealthy_threshold = 2
  }
}
이제 리스너 규칙을 생성해서 이 모든 부분을 연결해줘여 한다. 아래는 모든 경로에 대해서 위에서 만든 대상 그룹으로 보내는 리스너 규칙을 추가
resource "aws_lb_listener_rule" "alb_listener" {
  listener_arn = aws_lb_listener.http.arn
  priority = 100

  condition {
    path_pattern {
      values = ["*"]
    }
  }
  action {
    type = "forward"
    target_group_arn = aws_lb_target_group.alb_tg.arn
  }
}
```

## Terraform State management

테라폼을 apply 할때마다 테라폼은 생성한 인프라에 대한 정보를 테라폼 상태 파일에 기록한다. 파일은 .tfstate라는 파일로 생성이 되고 파일 내용은 json 형태로 되어있다. 이 파일은 인프라에 대한 정보를 담고 있기 때문에 함부로 파일을 open해서는 안되고 여러 사용자가 테라폼을 사용할 경우 공유 저장소에 테라폼의 상태파일을 관리해야한다. 만약 상태 파일을 조작해야 하는 경우가 있다면 terraform import or terraform state 명령을 사용해야한다.
</br></br>

테라폼을 실제 운영 환경에서 팀 단위로 사용할 경우 공유 스토리지와 상태 파일 잠금, 상태 파일 격리에 대한 문제에 접근하게 되는데 이는 테라폼에 내장된 원격 백엔드 기능을 사용하면 해결된다. 테라폼 백엔드는 테라폼이 상태를 로드하고 저장하는 방법을 결정하는데 만약 terraform apply 명령을 실행한다면 테라폼은 자동으로 잠금을 활성화 하게 된다. 물론 -lock-timeout을 통해 잠금 시간을 설정하거나 -lock=false를 통해 잠금을 해제할 수 있다. 그리고 기본적으로 원격 백엔드는 데이터를 보내거나 상태 파일을 저장할 때 암호화하는 것을 지원하는데 s3버킷과 iam 정책을 사용하면 액세스 제한을 구성할 수 있어서 파일에 포함된 스크릿이나 상태 파일 액세스에 대해 효율적으로 관리할 수 있다. 보통 s3에 상태 파일을 저장하고 dynamodb를 이용해서 잠금 기능을 지원한다.
</br>
원격 백엔드를 쓰기 앞서 먼저 사용할 s3와 잠금 기능을 활용할 dynamodb를 먼저 배포해야한다. force_destroy 변수는 나중에 삭제할때 추가해줘도 되는 부분이다(만약 버킷 안에 파일이 있으면 버킷이 삭제가 안되기 때문). 그리고 기존에 s3 bucket 안에 넣었던 version 같은 변수들이 deprecate 될거라고 권장되지 않아서 다른 리소스로 빼서 사용을 진행하였다.
```HCL
#아래에서 prevent_destroy를 true로 한 이유는 해당 버킷은 상태 파일을 위한 저장소이므로 삭제를 방지하기 위해서 설정해줬다.
resource "aws_s3_bucket" "example" {
  bucket = "terraform-example-s3-sumin"
  force_destroy = true
  # 삭제 방지
  lifecycle {
    prevent_destroy = true
  }
  # deprecated 된 내용들 (https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/s3_bucket)
#   versioning {
#     enabled = true
#   }
#   server_side_encryption_configuration {
#     rule {
#       apply_server_side_encryption_by_default {
#         sse_algorithm = "AES256"
#       }
#     }
#   }
}

#s3 버킷에 버전 관리를 활성해서 버킷의 파일이 업데이트될 때마다 새 버전을 만들어서 이전 버전으로 롤백할 수 있게 하였다.
resource "aws_s3_bucket_versioning" "example" {
  bucket = aws_s3_bucket.example.id
  versioning_configuration {
    status = "Enabled"
  }
}

#s3 버킷에 기록된 모든 데이터에 서버 측 암호화를 설정
resource "aws_s3_bucket_server_side_encryption_configuration" "example" {
  bucket = aws_s3_bucket.example.id

  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}
#기본 키를 LockID로 하는 테이블 생성.
resource "aws_dynamodb_table" "example" {
  name = "terraform-example-dynamodb-sumin"
  billing_mode = "PAY_PER_REQUEST"
  hash_key = "LockID"

  attribute {
    name = "LockID"
    type = "S"
  }
}
```
이렇게 tf 파일에 작성한 후 init을 통해 공급자 코드를 다운로드한 후 apply로 배포하면된다. 그 다음 backend 내용 작성한 후 init을 다시 해주면 상태 파일이 이젠 로컬이 아니라 s3에 저장되는 것을 확인할 수 있다.
```HCL
terraform {
  backend "s3" {
    bucket = "terraform-example-s3-sumin"
    key = "global/s3/terraform.tfstate"
    region = "us-east-2"

    dynamodb_table = "terraform-example-dynamodb-sumin"
    encrypt = true
  }
}
```
하지만 이러한 테라폼 백엔드는 삭제 절차나 생성 절차나 까다롭고 변수나 참조 같은 것을 사용할 수 없다는 단점이 있다(variable 같은거). 즉, 값들을 다 수동으로 입력해야 하고 이 과정에서 오류가 발생할 수도 있다. 이러한 수동 입력 대신 hcl 파일에 변수를 저장하고 key값만 정적으로 넣어놓고 필요한 경우 `terraform init -backed-config=example.hcl` 명령어로 백엔드를 구성할 수 있다.(주석 부분을 다 hcl 파일로 이동.)
```HCL
terraform {
  backend "s3" {
    key = "global/s3/terraform.tfstate"
    # bucket = "terraform-example-s3-sumin"
    # key = "global/s3/terraform.tfstate"
    # region = "us-east-2"

    # dynamodb_table = "terraform-example-dynamodb-sumin"
    # encrypt = true
  }
}
```
이렇게 파일을 추출하는 방법 말고도 [Terragrunt](https://terragrunt.gruntwork.io/docs/) 라는 오픈 소스 도구를 사용해서 기본 backend 설정을 하나의 파일에 정의하고 key 매개 변수를 모듈의 상대 경로에 설정하여 backend 구성을 반복하지 않도록 할 수 있다.</br></br>

원격 백엔드와 잠금을 사용하면 협업 과정에서 문제가 없지만 격리라는 문제가 남아있게 된다. 만약 상태 관리가 하나의 파일에서 되고 있다면 잘못됐을 경우 전체 인프라에 문제가 생길 수도 있다. 그래서 하나의 환경을 다른 환경으로부터 격리하는 과정이 필요하다. 상태 파일을 격리하는 방법은 크게 workspace를 통한 격리 방법과 파일 레이아웃을 이용한 격리 방법이 있다. 먼저 workspace부터 살펴보면 테라폼 상태를 별도의 이름을 가진 workspace로 분리해서 저장을 할 수 있는데 기본 작업 공간은 default에서 시작된다.
<br><br>
이전에 사용했던 코드로 s3버킷과 dynamodb를 생성하고 백엔드를 구성한 다음 `terraform worksapce show` 명령을 치면 현재 작업 공간을 확인할 수 있고 `terraform workspace list`를 통해선 작업 공간의 목록을 확인할 수 있다. 만약 새로운 작업 공간을 이용해서 상태파일을 관리하고 싶다면(인프라를 복제하는 작업 등을 할 경우) `terraform workspace new <name>` 명령을 통해 새 작업 공간을 만들고 이동할 수 있다. 이 상태에서 배포하려고 하는 코드를 apply하면 s3버킷에 env라는 폴더 내에 있는 <name>으로 된 폴더가 하나 만들어지고 여기 하위 아래에는 내가 key로 만들었던 형태 상태 파일이 관리가 된다. 만약 작업 공간을 바꾸고 싶을 때는 `terraform workspace select <name>`을 통해 작업 공간을 교체할 수 있다. 이러한 작업 공간을 변경하는 작업은 상태 파일이 지정된 경로를 변경하는 것과 동일하다. 하지만 이러한 작업 공간 분리 방식은 동일한 백엔드에 저장되게 된다는 단점이 있고 터미널에 현재 작업 공간에 대한 정보가 포함되어 있지 않기 때문에 유지 관리가 어렵다는 점이 있다.
<br><br>
파일 레이아웃을 이용한 격리 방식은 각 테라폼 구성 파일을 분리된 폴더에 넣는 방식이며 격리 수준을 구성 요소 수준으로 하는 방법이다. 
#### 일반적인 환경의 구성 요소별 폴더
- stage(테스트 환경과 같은 환경)
  - vpc
  - services
    - frontend
    - backend
      - variables.tf
      - outputs.tf
      - main.tf
  - data-storage
    - mysql
    - redis
- prod(production 환경)
  - vpc
  - services
    - frontend
    - backend
  - data-storage
    - mysql
    - redis
- mgmt(데브옵스 도구 환경)
  - vpc
  - services
    - jenkins
- global(모든 환경에서 사용되는 리소스들 s3나 iam 같은)
  - iam
  - s3

하지만 이렇게 파일 레이아웃을 분리하다보면 속성 참조가 어렵다는 점이 있는데 그대는 이제 terraform_remote_state 데이터 소스를 이용하면 된다. 이 데이터 소스를 사용하면 다른 테라폼 구성 세트에 완전한 읽기 전용 방식으로 저장된 테라폼 상태를 가져올 수 있다.
##### m1에서 init 시 template package가 없다는 에러가 뜰 경우 아래 명령어 입력
```bash
brew install kreuzwerker/taps/m1-terraform-provider-helper
m1-terraform-provider-helper activate
m1-terraform-provider-helper install hashicorp/template -v v2.2.0
```

먼저 기존의 s3와 ASG,ALB를 배포 해준다.(이때 s3 백엔드를 각각 추가) 그 다음 db의 데이터가 실제로 다른 파일에서 읽을 수 있는지 테스트 해본다.
mysql 폴더에 main.tf에 다음과 같은 코드를 넣어준다.
```HCL
resource "aws_db_instance" "example" {
  identifier_prefix = "terraform-example-db"
  engine = "mysql"
  allocated_storage = 10
  instance_class = "db.t2.micro"
  db_name = "exampleDb"
  username = "admin"
  manage_master_user_password   = true
  master_user_secret_kms_key_id = aws_kms_key.example.key_id
  skip_final_snapshot = true
}
resource "aws_kms_key" "example" {
  description = "mysql KMS Key"
}
```
책에서는 password를 설정하는 방법을 secrets manager에 키값을 저장해놓고 갖고오는 방식과 외부 환경 변수를 통해 갖고오는 방식 둘다 보여줬는데 기존은 secrets manager에 키를 만들지 않고 공식 문서에 나와있는대로 master 패스워드를 kms를 생성해서 secrets manager가 관리하는 방식을 선택했다.(https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/db_instance#delete)
<br><br>

또 다른 방법으로는 variables.tf에 dp_password라는 변수를 만들고 `export TF_VAR_db_password="(pass password)"`라는 명령어로 저장한다음 apply를 하면 패스워드를 외부 환경변수로부터 얻어서 사용할 수 있다.
<br><br>

데이터베이스쪽에서 백엔드를 이용해서 상태 정보를 s3에서 저장하고 있어서 이걸 다시 불러오도록 하기 위해선 terraform_remote_state를 사용하면 된다. 그리고 배쉬 스크립트가 길어지면 가독성이 떨어지기 때문에 이걸 외부화하면 더 편리하게 구성의 정의할 수 있다.(내장 함수와 template_file 데이터 소스를 이용)
```HCL
data "terraform_remote_state" "db" {
  backend = "s3"
  config = {
    bucket = "terraform-example-s3-sumin"
    key = "stage/data-stores/mysql/terraform.tfstate"
    region = "us-east-2"
   }
}
data "template_file" "user_data" {
  template = file("user-data.sh")

  vars = {
    server_port = var.server_port
    db_address = data.terraform_remote_state.db.outputs.address
    db_port = data.terraform_remote_state.db.outputs.port
  }
}
```

### 테라폼 모듈
위에 stage 환경과 prod환경에서는 동일한 코드를 사용하고 있고 이 코드들은 불필요하게 반복되고 있다. 그래서 이 코드들을 모듈로 만들어서 실제로 stage와 prod에서는 모듈을 호출해서 사용할 수 있도록 하면 코드의 재사용성을 높일 수 있다.

모듈의 파일 구조는 기존에 있던 파일 구조를 그대로 따라가면 된다.

- modules
  - services
    - webserver-cluster
      - main.tf
      - outputs.tf
      - user-data.sh
      - variables.tf

그리고 기존의 stage 코드는 다음과 같이 바꿔주면 된다.(아래는 하드 코딩된 부분을 variable로 바꿔서 적용한 부분이라 상이할 수 있음.)
```HCL
module "webserver_cluster" {
  source = "../../../modules/services/webserver-cluster/services/webserver-cluster"
  db_remote_state_bucket = "terraform-example-s3-sumin"
  db_db_remote_state_key = "stage/data-stores/mysql/terraform.tfstate"
  cluster_name = "webserver-stage"
  min_size = 2
  max_size = 2
  instance_type = "t2.micro"
}
```
이전 코드는 보안 그룹 이름이나 등등이 다 하드코딩되어 있어서 만약 모듈을 두번 실행하게 되면 중첩되는 오류가 발생해서 하드 코딩된 부분을 variable로 분리하고 variable에 있는 내용을 위와 같이 모듈을 사용하는 부분에서 입력을 받을 수 있도록 해야한다.
```HCL
variable "cluster_name" {
  description = "To use for all cluster resources"
  type = string
}
variable "db_remote_state_bucket" {
  description = "S3 bucket for the database's remote state"
  type = string
}
variable "db_remote_state_key" {
  description = "The path for the db remote state in S3"
  type = string
}
variable "instance_type" {
  description = "EC2 instances to run"
  type = string
}
variable "min_size" {
  description = "minimum number of EC2 instances in the ASG"
  type = number
}
variable "max_size" {
  description = "maximum number of EC2 instances in the ASG"
  type = number
}
```
이제 module 코드에서 하드코딩되어 있던 부분을 var.변수명으로 다 치환을 하면 된다. 위의 방식처럼 만약 module에서 따로 입력을 받을 필요가 있는 변수들이 있다면 module의 variable을 이용헤서 선언하면 되고 stage환경이나 prod환경에서 모듈을 사용할때 입력 변수로 대입해주면 된다. 하지만 이 모듈 내에서만 쓰고 모듈을 사용하는 쪽에서 구성 가능한 입력 변수로 노출하고 싶지 않은 경우가 있는데 이럴 때는 `local 변수`를 사용하면 된다.

```HCL
아래 코드는 module의 main.tf에 추가해주면 되고 하드코딩된 부분을 아래처럼 수정해주면 된다.
locals {
  http_port = 80
  any_port = 0
  any_protocol = "-1"
  tcp_protocol = "tcp"
  all_ips = ["0.0.0.0/0"]
}
resource "aws_security_group_rule" "all_outbound" {
  type = "engress"
  security_group_id = aws_security_group.lb_example_sg.id
  cidr_blocks = local.all_ips
  from_port = local.any_port
  protocol = local.any_protocol
  to_port = local.any_port
}
```

prod 환경의 경우 특정 시간에는 트래픽이 평균적으로 많고 특정 시간이 지나면 트래픽이 평균적으로 낮아지는데 이럴때 autoscaling_schedule 리소스를 이용하면 된다.
```HCL
아래 시간은 cron에 등록하는 시간과 동일하며 순서대로 분 시 일 월 년 순이고 아래의 코드는 사용하는 prod의 main.tf에 추가해주면 된다. module.webserver_cluster.asg_name
이 부분은 module에서 output.tf에서 선언해준 것을 이용하면 된다.
resource "aws_autoscaling_schedule" "scale_out_during_business_hours" {
  scheduled_action_name = "scale-out-during-business-hours"
  min_size = 2
  max_size = 10
  desired_capacity = 10
  recurrence = "0 9 * * *"
  autoscaling_group_name = module.webserver_cluster.asg_name
}
resource "aws_autoscaling_schedule" "scale_in_at_night" {
  scheduled_action_name = "scale-in-at-night"
  min_size = 2
  max_size = 10
  desired_capacity = 2
  recurrence = "0 17 * * *"
  autoscaling_group_name = module.webserver_cluster.asg_name
}
```

모듈에서 주의해야할 점 중 하나는 파일 경로이다. 모듈에서 file 함수를 사용할 때는 파일 경로를 상대 경로를 사용해야 한다.
테라폼이 제공하는 파일 유형 타입은 총 세가지이다.(https://developer.hashicorp.com/terraform/language/expressions/references#filesystem-and-workspace-info)
- path.module : 표현식이 정의된 모듈의 파일 시스템 경로를 반환. 하지만 이건 외부 모듈인지 로컬 모듈인지에 따라 다른 동작이 되기 때문에 쓰기 작업엔 사용하지 않는게 좋다.
- path.root : 루트 모듈의 파일 시스템 경로를 반환.
- path.cwd : 현재 작업 중인 디렉토리의 파일 시스템 경로를 반환. 일반적으로 path.root와 동일하지만 이외의 다른 디렉토리를 쓸 때는 경로가 달라진다.

또한, 리소스 구성을 인라인 블록으로 하기보단 별도의 리소스로 구분하는게 좋은데 그 이유는 모듈의 유연성을 위해서이다. 예를 들어 현재 구현한 보안그룹 규칙을 ingress랑 engress를 하나의 리소스에 작성했는데 아래처럼 분리하면 모듈의 유연성을 확보할 수 있다.
```HCL
resource "aws_security_group_rule" "http_inbound" {
  type = "ingress"
  security_group_id = aws_security_group.lb_example_sg.id
  cidr_blocks = local.all_ips
  from_port = local.http_port
  protocol = local.tcp_protocol
  to_port = local.http_port
}
resource "aws_security_group_rule" "all_outbound" {
  type = "engress"
  security_group_id = aws_security_group.lb_example_sg.id
  cidr_blocks = local.all_ips
  from_port = local.any_port
  protocol = local.any_protocol
  to_port = local.any_port
}
```

그리고 git같은 걸 이용해서 모듈의 버전 관리도 하면 인프라 구성이 잘못되었을 경우 이전 버전으로 롤백하거나 그럴 수 있다. 또한, 모듈을 사용할 때 source에 url을 이용해서도 사용이 가능하고 ref통해 버전 관리도 가능하다.(tag이용)
```HCL
module "webserver_cluster" {
  source = "github.com/brikis98/terraform-up-and-running-code//code/terraform/04-terraform-module/module-example/modules/services/webserver-cluster?ref=v0.1.0"
  db_remote_state_bucket = "terraform-example-s3-sumin"
  db_db_remote_state_key = "prod/data-stores/mysql/terraform.tfstate"
  cluster_name = "webserver-prod"
  min_size = 2
  max_size = 10
  instance_type = "m4.large"
}
```