# Terraform Concept

[https://github.com/hashicorp/terraform](https://github.com/hashicorp/terraform)

[https://developer.hashicorp.com/terraform/intro](https://developer.hashicorp.com/terraform/intro)

테라폼은 HashiCorp에서 만든 코드형 인프라 도구인데 이걸 이용해서 클라우드 및 온프레미스 리소스를 모두 정의할 수 있고 버전화나 재사용 및 공유도 가능.

테라폼을 이용하면 워크플로를 사용해서 수명 주기 동안 모든 인프라를 프로비저닝하고 관리할 수 있음.

컴퓨팅, 스토리지, 네트워크 리소스와 같은 low-level의 리소스 뿐만 아니라 DNS나 SaaS 기능과 같은 high-level 리소스도 관리할 수 있음.

## 작동 방식

테라폼은 API를 통해 클라우드 플랫폼 및 기타 서비스에서 리소스를 만들고 관리. 테라폼과 target api 사이에 terraform provider라는게 존재하고 aws, azurue 등이 terraform registry에서 공개적으로 공급자를 제공하고 있음.

테라폼의 workflow는 세 단계로 구성

- write - cloud provider나 서비스를 거치는 리소스를 정의(ex: vpc 설정)
- plan - 기존 인프라 및 구성을 기반으로 생성, 업데이트 또는 destroy할 인프라를 describing 하는 실행 plan을 생성
- apply - apply 시 테라폼은 모든 리소스 종속성을 고려하여 올바른 순서로 제안된 작업을 수행

## 인프라 추적

terraform은 plan을 생성하고 인프라를 수정하기 전에 apply를 요청한다. 또한 환경에 대한 정보 소스 역할을 하는 state file에서 실제 인프라를 추적하며 이를 토대로 구성과 일차하도록 인프라에 대한 변경 사항을 결정한다.

## 변경 자동화

테라폼의 구성 파일은 선언적이면 리소스를 만들기 위한 단계적 지침을 작성할 필요없이 기본 logic을 처리해준다. 테라폼은 자원 그래프를 작성해서 자원 종속성을 판별하고 비종속 자원을 병렬로 생성하거나 수정한다.

## 구성 표준화

테라폼은 구성 가능한 인프라 컬렉션을 정의하는 module이라는 재사용 가능한 구성 요소를 지원한다. 테라폼 레지스트리에서 공개적으로 사용 가능한 모듈을 사용하거나 직접 작성할 수 있다.


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