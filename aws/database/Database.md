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