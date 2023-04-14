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