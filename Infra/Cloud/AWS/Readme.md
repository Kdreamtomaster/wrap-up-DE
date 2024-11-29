
# AWS 클라우드 개념 정리

## 목차
1. [AWS Identity and Access Management (IAM)](#aws-identity-and-access-management-iam)
2. [Amazon EC2 및 Auto Scaling](#amazon-ec2-및-auto-scaling)
3. [Amazon S3 및 Glacier](#amazon-s3-및-glacier)
4. [AWS Storage Gateway](#aws-storage-gateway)
5. [AWS Networking 및 라우팅](#aws-networking-및-라우팅)
6. [Amazon CloudFront](#amazon-cloudfront)
7. [AWS 데이터 처리 및 분석](#aws-데이터-처리-및-분석)

---

## AWS Identity and Access Management (IAM)

### **1. 교차 계정 액세스(Cross-Account Access)**
- **IAM 역할(Role)**을 사용해 여러 AWS 계정 간의 리소스 접근을 제어.
- **구성 방법**:
  1. **Master 계정**:
      - IAM 사용자 생성.
      - 사용자는 역할을 Assume(위임)하여 다른 계정 리소스에 접근 가능.
  2. **Dev/Test 계정**:
      - IAM 역할(Role)을 생성하여 Master 계정에서 사용 가능하도록 설정.
      - 역할에 Admin 권한을 부여하여 Dev/Test 계정의 리소스를 관리.

- **AWS 모범 사례**:
  - IAM 역할(Role)을 통해 교차 계정 액세스를 제공.
  - Consolidated Billing은 비용 관리만 제공하며, 권한은 별도 설정 필요.

---

## Amazon EC2 및 Auto Scaling

### **1. Reserved Instances(RI) 및 Auto Scaling**
- **Reserved Instances(RI)**:
  - EC2 인스턴스의 장기 사용 시 비용 절감.
  - Heavy utilization, Medium utilization 등의 옵션 제공.
- **Auto Scaling 그룹**:
  - 트래픽 부하에 따라 EC2 인스턴스를 동적으로 추가/제거.
  - ELB(Elastic Load Balancer)와 연동하여 부하를 분산.

### **2. ELB와 Route 53**
- ELB는 여러 EC2 인스턴스에 트래픽을 분산.
- Route 53의 **가중치 기반 라우팅(Weighted Routing)**:
  - 여러 ELB 간 트래픽을 비율에 따라 분산.
  - 다양한 인스턴스 유형(RI 포함)을 효율적으로 활용.

---

## Amazon S3 및 Glacier

### **1. Amazon S3**
- **객체 스토리지**:
  - 데이터를 개별 객체로 저장하며, POSIX 호환이 아님.
  - 주요 사용 사례: 데이터 백업, 정적 웹 호스팅.
- **중요 특징**:
  - 다양한 스토리지 클래스 제공: Standard, IA(Infrequent Access), Glacier.
  - 트래픽 부하에 따라 객체 캐싱 가능.

### **2. Amazon Glacier**
- **장기 보관용 저비용 스토리지**:
  - 비용 효율적이지만 데이터 복구 시간이 느림.
  - 실시간 데이터 접근이 필요 없는 백업에 적합.

---

## AWS Storage Gateway

### **1. Gateway Cached Volumes**
- 데이터를 AWS S3에 저장하면서 자주 사용하는 데이터는 온프레미스에 캐싱.
- **장점**:
  - 온프레미스 스토리지 부담 감소.
  - POSIX 호환 블록 기반 스토리지 제공.
- **적합한 사용 사례**:
  - 대규모 데이터 백업 및 백업 중 데이터 일부 접근.

### **2. Gateway Stored Volumes**
- 온프레미스에 데이터의 전체 복사본 저장.
- AWS S3에 백업 복사본 저장.
- **단점**:
  - 로컬 스토리지 용량 요구가 크며, 대규모 데이터에 적합하지 않음.

---

## AWS Networking 및 라우팅

### **1. EBS-Optimized 네트워크**
- EC2와 EBS 간의 전용 대역폭 제공.
- 병목 현상 해결:
  - 대역폭 제한이 EBS IOPS를 충분히 처리하지 못할 때 발생.
  - 더 높은 대역폭을 지원하는 EC2 인스턴스로 업그레이드 필요.

### **2. Route 53**
- **라우팅 정책**:
  - 지연 시간 기반 라우팅(Latency-Based Routing): 가장 빠른 리전으로 트래픽 라우팅.
  - 가중치 기반 라우팅(Weighted Routing): 여러 리소스 간 트래픽 분산.

---

## Amazon CloudFront

### **1. 캐시 최적화**
- **캐시 히트 비율(Cache Hit Ratio)**:
  - 쿼리 문자열의 순서와 대소문자 불일치로 캐시 히트 비율이 낮아질 수 있음.
- **Lambda@Edge 활용**:
  - 쿼리 문자열을 정렬하고 소문자화하여 캐시 키를 일관성 있게 유지.
  - 캐시 히트 비율을 효과적으로 증가.

---

## AWS 데이터 처리 및 분석

### **1. Amazon Kinesis**
- **실시간 데이터 스트리밍**:
  - 로그 데이터를 실시간으로 수집 및 처리.
  - 데이터의 샘플링 및 분석에 적합.
- **적합한 사용 사례**:
  - 실시간 로그 분석 및 데이터 보존(12시간 내).

### **2. Amazon S3 및 EMR**
- **S3**:
  - 대규모 데이터 저장.
- **EMR**:
  - 배치 데이터 처리 및 분석.
- **단점**:
  - 실시간 데이터 처리에는 적합하지 않음.

---

### **정리**
- AWS의 다양한 서비스는 각각의 용도와 요구사항에 맞게 설계되었습니다.
- 이 정리는 문제 풀이를 기반으로 AWS 클라우드 아키텍처와 활용법을 체계적으로 정리한 자료입니다.
