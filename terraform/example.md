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