# 2.1 Amazon VPC를 사용해 프라이빗 가상 네트워크 생성
해당 리전에 vpc를 만들고 그에 대한 CIDR(classless inter-domain routing) 블록 구성

``` bash
# ipv4 CIDR 블록을 가진 VPC 생성
VPC_ID=$(aws ec2 create-vpc --cidr-block 10.10.0.0/16 --tag-specifications 'ResourceType=vpc, Tags=[{Key=Name, Value=aws201}]' --output text --query Vpc.VpcId)

# VPC 상태 확인
aws ec2 describe-vpcs --vpc-ids $VPC_ID

# CIDR 블록을 한번 VPC와 연결하면 확장할 수는 있지만 수정은 불가능. IPv4 공간 추가
aws ec2 describe-vpcs --vpc-ids $VPC_ID

# VPC를 ipv6으로 만들기
aws ec2 create-vpc --cidr-block 10.11.0.0/16 --amazon-provided-ipv6-cidr-block --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name, Value=aws201-ipv6}]'
```

# 2.2 서브넷과 라우팅 테이블 포함한 네트워크 티어 생성
리소스의 분할 및 중복을 위해 개별 ip 공가능로 구성한 vpc 네트워크 생성 챕터.
 하나의 vpc에 있는 2개의 AZ에 각각 subnet을 하나씩 생성하고 이를 라우팅 테이블을 이용해서 트래픽을 처리하는 방식.
 ``` bash
 # vpc는 이전 챕터에서 만든걸 이용하고 먼저 라우팅 테이블 생성
 ROUTE_TABLE_ID_1=$(aws ec2 create-route-table --vpc-id $VPC_ID --tag-specifications 'ResourceType=route-table, Tags=[{Key=Name,Value=aws202}]' --output text --query RouteTable.RouteTableId)

 # 각 AZ에 서브넷을 생성
 SUBNET_ID_1=$(aws ec2 create-subnet --vpc-id $VPC_ID --cidr-block 10.10.0.0/24 --availability-zone us-west-2a --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=aws202a}]' --output text --query Subnet.SubnetId)

 SUBNET_ID_2=$(aws ec2 create-subnet --vpc-id $VPC_ID --cidr-block 10.10.1.0/24 --availability-zone us-west-2b --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=aws202b}]' --output text --query Subnet.SubnetId)

 # AZ의 매개변수로 리전 뒤에 a나 b를 추가해서 각 서브넷을 프로비저닝할 논리적 AZ를 지정. AWS에서는 AZ 간에 리소스 균형을 맞추고자 계정별로 이름과 가용 영역을 무작위로 지정.

 # 라우팅 테이블을 서브넷과 연결
 aws ec2 associate-route-table --route-table-id $ROUTE_TABLE_ID --subnet-id $SUBNET_ID_1

 aws ec2 associate-route-table --route-table-id $ROUTE_TABLE_ID --subnet-id $SUBNET_ID_2

 # 두 서브넷의 AZ과 라우팅 테이블에 대해 확인하는 코드
 aws ec2 describe-subnets --subnet-ids $SUBNET_ID_1
 aws ec2 describe-subnets --subnet-ids $SUBNET_ID_2
 aws ec2 describe-route-tables --route-table-ids $ROUTE_TABLE_ID

 # 서브넷 설계할 때는 현재 요구사항에 맞는 서브넷 크기를 선택해야함(너무 크게 설정해서도 안되고 너무 적게 설정해서도 안됨. 너무 크면 리소스가 낭비되고 작으면 리소스가 부족하게 됨).
 # 리전에서 VPC를 생성해라 때 해당 네트워크 계층의 AZ에 서브넷을 분산하는 것이 모범사례. 보통 3개 이상의 AZ를 갖고 있고 2개를 쓰는곳도 있다고는 들음. 보통 2-3개가 일반적인 사례인것 같음.
 ```

 # 2.3 인터넷 게이트웨이를 사용해 VPC를 인터넷에 연결

``` bash
# IGW 생성
INET_GATEWAY_ID=$(aws ec2 create-internet-gateway --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=aws203}]' --output text --query InternetGateway.InternetGatewayId)

# IGW를 기존 VPC에 연결
aws ec2 attach-internet-gateway --internet-gateway-id $INET_GATEWAY_ID --vpc-id $VPC_ID

# 새로운 라우팅 테이블을 만들고 해당 라우팅 테이블을 서브넷에 연결
aws
 ROUTE_TABLE_ID_2=$(aws ec2 create-route-table --vpc-id $VPC_ID --tag-specifications 'ResourceType=route-table, Tags=[{Key=Name,Value=aws203b}]' --output text --query RouteTable.RouteTableId)
# VPC의 각 라우팅 테이블에서 기본 경로 대상을 인터넷 게이트웨이로 설정하는 경로 생성(IGW와 연결된 경로가 0.0.0.0/0인 서브넷은 퍼블릿 서브넷으로 간주)
aws ec2 create-route --route-table-id $ROUTE_TABLE_ID_1 --destination-cidr-block 0.0.0.0/0 --gateway-id $INET_GATEWAY_ID

aws ec2 create-route --route-table-id $ROUTE_TABLE_ID_2 --destination-cidr-block 0.0.0.0/0 --gateway-id $INET_GATEWAY_ID

# EIP(탄력적 ip) 생성
ALLOCATION_ID=$(aws ec2 allocate-address --domain vpc --output text --query AllocationId)

# EIP를 기존 EC2 인스턴스에 연결
aws ec2 associate-address --instance-id $INSTANCE_ID --allocation-id $ALLOCATION_ID

# 그 다음 SSM session manager를 사용해 EC2 인스턴스에 연결해야하는데 또 잘안돼서 그냥 ssh로 들어가서 따라함.

```

# 2.4 NAT 게이트웨이 사용한 프라이빗 세브넷의 외부 인터넷 접근
``` bash
# 먼저 이전에 만들었던 방식으로 프라이빗 서브넷을 생성하고 라우팅 테이블을 서브넷에 연결
# NAT 게이트웨이 사용할 탄력적 ip 주소 생성
ALLOCATION_ID=$(aws ec2 allocate-address --domain vpc --output text --query AllocationId)

# AZ2의 퍼블릭 서브넷 내에 nat 게이트웨이 생성
NAT_GATEWAY_ID=$(aws ec2 create-nat-gateway --subnet-id $SUBNET_ID_2 --allocation-id $ALLOCATION_ID --output text --query NatGateway.NatGatewayId)

# 상태 확인(콘솔로 보는게 나을듯)
aws ec2 describe-nat-gateways --nat-gateway-ids $NAT_GATEWAY_ID --output text --query NatGateways.State

# nat 게이트웨이의 0.0.0.0/0에 대한 기본 경로를 프라이빗 서브네쪽 라우팅 테이블에 추가
aws ec2 create-route --route-table-id $ROUTE_TABLE_ID_3 --destination-cidr-block 0.0.0.0/0 --nat-gateway-id $NAT_GATEWAY_ID

aws ec2 create-route --route-table-id $ROUTE_TABLE_ID_4 --destination-cidr-block 0.0.0.0/0 --nat-gateway-id $NAT_GATEWAY_ID

# 이후 인스턴스에 연결해서 잘되는지 확인
```
이렇게 하나의 AZ에 nat 게이트웨이를 프로비저닝하는 것보다 각 AZ에 프로비저닝하는게 AZ 트리팩 양을 줄이는데 좋고 탄력성 있어짐.
# 2.5 보안 그룹을 참고해 동적으로 접근 권한 부여
``` basg
# 보안 그룹 생성
SG_ID=$(aws ec2 create-security-group --group-name aws205sg --description "Instance Security Group" --vpc-id $VPC_ID --output text --query GroupId)

# 보안 그룹 인스턴스에 연결
aws ec2 modify-instance-attribute --instance-id $INSTANCE_ID_1 --groups $SG_ID
aws ec2 modify-instance-attribute --instance-id $INSTANCE_ID_2 --groups $SG_ID


# tcp 22에 대한 액세스를 허용하는 인바운드 규칙 추가(이렇게 하니 콘솔으로는 ec2인스턴스에 접근이 안됨.. EIP도 주고 했지만 잘안돼서 인바운드 규칙을 0.0.0.0으로 수정하고 진행함. )
aws ec2 authorize-security-group-ingress --protocol tcp --port 22 --source-group $SG_ID --group-id $SG_ID

# 이후 ssm으로 접근해서 다른 인스턴스에 접근하는거지만 ssm 연결이 잘되지도 않고 그래서 EIP를 인스턴스에 연결해서 직접 콘솔로 접근해서 진행.
```

# 2.6 VPC Reachability Analyzer를 활용한 네트워크 경로 확인 및 문제 해결

``` bash
# 시작하기 앞서 사용자 정책에 reachability analyzer 정책을 추가. 안될 경우엔 정책을 인라인으로 추가.

# ec2인스턴스와 tcp 22 지정하는 네트워크 인사이트 경로 생성
INSIGHTS_PATH_ID=$(aws ec2 create-network-insights-path \
    --source $INSTANCE_ID_1 --destination-port 22 \
    --destination $INSTANCE_ID_2 --protocol tcp \
    --output text --query NetworkInsightsPath.NetworkInsightsPathId)

# 두 인스턴스 간의 네트워크 인사이트 분석
ANALYSIS_ID_1=$(aws ec2 start-network-insights-analysis \
    --network-insights-path-id $INSIGHTS_PATH_ID --output text \
    --query NetworkInsightsAnalysis.NetworkInsightsAnalysisId)
aws ec2 describe-network-insights-analyses \
    --network-insights-analysis-ids $ANALYSIS_ID_1
#networkpathFound가 false인것을 확인.

# 인스턴스 2에 연결된 보안 그룹 업데이트. 1의 보안그룹에서 tcp 22로의 액세스를 허용하는 규칙 추가.
aws ec2 authorize-security-group-ingress \
   --protocol tcp --port 22 \
   --source-group $SG_ID \
   --group-id $SG_ID_2

# 인사이트 분석 다시 실행
ANALYSIS_ID_2=$(aws ec2 start-network-insights-analysis \
    --network-insights-path-id $INSIGHTS_PATH_ID --output text \
    --query NetworkInsightsAnalysis.NetworkInsightsAnalysisId)
aws ec2 describe-network-insights-analyses \
    --network-insights-analysis-ids $ANALYSIS_ID_2

# true로 바뀐 것을 확인할 수 있음. 이후엔 전의 방식대로 진행
```
네트워크 연결을 네트워크 인사이트 경로로 정의해 테스트할 수 있음. 처음에는 ssh연결이 허용되지 않았기 때문에 나중에 보안그룹을 업데이트하고 나서 사용할 수 있음. VPC reachability analyzer는 인프라를 프로비저닝할 필요가 없기 때문에 서버리스 방식으로 네트워킁 문제 해결 및 구성의 유효성 검증을 위한 효율적인 도구.

# 2.7 Application Load Balancer를 사용해 HTTP 트래픽을 HTTPS로 리디렉션
