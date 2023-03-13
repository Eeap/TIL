# 4.1 Amazon Aurora Serverless PostgreSQL
Amazon Aurora는 Aurora Serverless v2, 메모리 최적화 및 버스트 가능 성능이라는 3가지 DB 인스턴스 클래스 유형을 제공

db.serverless - Aurora Serverless v2에서 사용하는 특수 DB 인스턴스 클래스. Aurora는 워크로드의 변화에 따라 컴퓨팅, 메모리 및 네트워크 리소스를 동적으로 조정.

``` bash
# secretmanager 이용해서 암호 생성
ADMIN_PASSWORD=$(aws secretsmanager get-random-password --exclude-punctuation --password-length 41 --require-each-included-type --output text --query RandomPassword)

# 데이터베이스 서브넷 그룹 생성(RDS ENI 그룹을 단순화하기 위함)
aws rds create-db-subnet-group --db-subnet-group-name aws401subnet --db-subnet-group-description "aws401 subnet group" --subnet-ids subnet-1 subnet-2

# 데이터베이스 vpc 보안 그룹 생성
DB_SECURITY_GROUP_ID=$(aws ec2 create-security-group --group-name aws401sg --description "aurora serverless security group" --vpc-id vpc-id --output text --query GroupId)

# engine-mode를 serverless로 데이터베이스 클러스터 생성
aws rds create-db-cluster --db-cluster-identifier aws401dbcluster --engine aurora-postgresql --engine-mode serverless --engine-version 10.14 --master-username dbadmin --master-user-password $ADMIN_PASSWORD --db-subnet-group-name aws401subnet --vpc-security-group-ids $DB_SECURITY_GROUP_ID

# Autoscaling 용량 목표 설정
aws rds modify-db-cluster --db-cluster-identifier aws401dbcluster --scaling-configuration MinCapacity=8,MaxCapacity=16,SecondsUntilAutoPause=300,TimeoutAction='ForceApplyCapacityChange',AutoPause=true
```

# 4.2 IAM 인증을 사용한 RDS 접속
aws를 이용한 db 접근 방법 중 IAM을 이용한 방법!
```
코드 대신 과정 나열.(콘솔로 진행)
먼저 RDS 데이터베이스의 인스턴스의 IAM 데이터베이스 인증을 활성화(처음 생성시 설정 가능) -> ec2 인스턴스 역할에 rds에 접근할 수 있는 정책 추가 -> 그 다음 IAM 인증 시 사용할 새로운 데이터베이스 사용자 생성 -> RDS 인증 토큰을 생성하고 ec2에서 mysql의 명령어로 접근할 때 토큰을 password로 쓰고 aws에서 제공하는 rds CA파일을 다운로드해서 접근
```
IAM은 임시 토큰을 15분 동안 유지하고 권한 부여는 데이터베이스 사용자로 부여!

# 4.3 RDS 프록시를 사용한 람다와 RDS 연결
이번 챕터도 cli 대신 콘솔로 진행
```
들어가기 앞서 커넥션 풀링을 사용한다고 했는데 커넥션 풀링이란 데이터베이스와 연결된 커넥션을 미리 만들어 놓고 이를 pool로 관리하는 방식. 이렇게 커넥션 풀링 방식을 이용하면 연결 수를 최소화할 수 있고 성능을 향상시킬 수 있음. RDS프록시는 연결을 pool로 관리함.

mysql rds를 생성하고 기다린 다음 proxy를 생성하면됨. IAM은 rds 관련으로 선택! 그 다음 RDS 프록시를 만들면 기본적으로 endpoint가 만들어지고 대상그룹을 지정할 수 있는데 db 인스턴스로 지정. RDS 프록시를 이용하는 vpc은 보안그룹에 db로 향하는 3306포트에 대해 허용하고 람다함수도 간단하게 db proxy를 지정할 수 있는 설정이 구성칸에 존재. (추가적인 과정이 필요하지만 초기에 db 인스턴스 버전을 잘못 선택해서 proxy로 대상 선택이 안돼서 여기까지는 구성완료!) 추후에 필요한 과정이 단순 보안그룹 설정과 연결 확인 과정

rds에 직접 접근하는 것보다 proxy로 관리하면 커넥션도 관리할 수 있고 성능 향상을 기대할 수 있음.
```
