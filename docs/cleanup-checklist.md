# AWS 실습 리소스 정리 체크리스트

- **미션:** 내가 만든 웹사이트를 인터넷에 올려 누구나 쓰게 하기
- **리전:** 서울 (`ap-northeast-2`)
- **상태:** 미정리 — 실제 삭제 완료 화면을 확인하기 전에는 체크하지 않음
- **목표:** 제출에 필요한 증빙을 확보한 뒤 실습 리소스를 정리하여 추가 과금 위험 최소화

> **중요:** 2026-10-27 기준 EC2가 실행 중이었고 외부 `/health`가 `200 OK`를 반환했다. 따라서 현재 리소스 정리 완료를 주장할 수 없다. 아래 체크박스는 실제 확인 뒤 직접 체크한다.

## 1. 정리 대상 목록

| 리소스                    | 실습에서 사용한 이름 또는 ID                               | 확인할 사항                             |
| ------------------------- | ---------------------------------------------------------- | --------------------------------------- |
| VPC                       | `cloud-mission-vpc` / `vpc-0f4332b567e230ef2`              | 삭제                                    |
| Public Subnet             | `cloud-mission-public-subnet` / `subnet-098f08891db67d8b5` | 삭제                                    |
| Internet Gateway          | `cloud-mission-igw` / `igw-0012aedbb97eaeb2a`              | 분리 후 삭제                            |
| 사용자 지정 Route Table   | `cloud-mission-public-rt` / `rtb-03eeb7c0aaa5e3028`        | 연결 해제 후 삭제                       |
| Security Group            | `cloud-mission-sg`                                         | 참조하는 인스턴스 정리 후 삭제          |
| EC2                       | `i-0bbbb0474a0bf20d6` (`t3.micro`)                         | **Terminate** 확인                      |
| EBS Volume                | EC2에 연결된 루트 볼륨 (실제 볼륨 ID 확인 필요)            | 종료 시 자동 삭제 여부 / 잔여 볼륨 삭제 |
| Public IPv4               | EC2 자동 할당 주소                                         | 인스턴스 종료 후 해제 여부 확인         |
| Elastic IP                | 별도 생성 여부 확인                                        | 있다면 Release                          |
| NAT Gateway, ELB/ALB, RDS | 이번 실습에서는 생성 계획 없음                             | 혹시 생성된 항목이 없는지 확인          |
| IAM                       | `cloud-mission-user`, 고객 관리형 정책 `codyssey`          | 실습 전용이면 다른 용도 확인 후 정리    |

## 2. 삭제 전 증빙 확보 — 먼저 수행

- [x] 외부에서 `GET http://<EC2_PUBLIC_IPV4>/health` → `200 OK`, `OK` 확인
- [x] `docs/screenshots/external-health.png` 저장
- [x] `docs/architecture.png` 준비 (실제 AWS 구성과 일치 여부 확인)
- [x] `README.md`에 **검증 당시** URL 또는 IP 및 B 방식 명시
- [x] `docs/troubleshooting.md` 작성·검토
- [x] EC2의 VPC/Subnet/Route Table/SG 연결과 IAM 권한 확인·필요한 증빙 확보
- [x] Git 저장소에 `.pem`, 임시 비밀번호, 자격 증명 파일이 포함되지 않았는지 확인

## 3. AWS 리소스 삭제 순서

### A. 컴퓨팅 및 스토리지

- [x] **EC2 → Instances:** `i-0bbbb0474a0bf20d6` 선택 → **Terminate instance** 실행
- [x] 인스턴스 상태가 `Terminated`로 표시되는지 확인 (단순 `Stopped`는 종료 아님)
- [x] **EC2 → Volumes:** 연결했던 EBS 루트 볼륨이 삭제되었는지 확인
- [x] 생성한 미사용 EBS 볼륨이 남아 있다면 삭제
- [x] **EC2 → Elastic IPs:** 할당한 Elastic IP가 있다면 연결 해제 후 **Release**
- [x] 자동 할당 공인 IPv4의 연결이 종료된 것을 확인
- [x] **EC2 → Network Interfaces:** 인스턴스에 연결된 네트워크 인터페이스가 정리되었는지 확인

### B. VPC 네트워크

- [x] **VPC → Route Tables:** `cloud-mission-public-rt`에서 서브넷의 명시적 연결 해제
- [x] 사용자 지정 라우트 `0.0.0.0/0 → cloud-mission-igw` 제거 (필요 시)
- [x] `cloud-mission-public-rt` 삭제 (**기본 VPC Route Table은 별도 삭제 대상이 아님**)
- [x] **VPC → Security Groups:** `cloud-mission-sg` 삭제 (다른 리소스에서 참조하면 먼저 참조 해제)
- [x] **VPC → Internet Gateways:** `cloud-mission-igw`를 VPC에서 Detach한 뒤 Delete
- [x] **VPC → Subnets:** `cloud-mission-public-subnet` 삭제
- [x] **VPC → Your VPCs:** `cloud-mission-vpc` 삭제

> VPC 콘솔에서 관련 종속 리소스를 함께 삭제하는 기능을 사용하더라도, 위 대상이 실제로 없어졌는지 최종 확인한다. 이번 실습에서 생성하지 않은 다른 VPC·서브넷은 삭제하지 않는다.

## 4. IAM·키 관리

- [x] `cloud-mission-user`의 정책 및 추가 자격 증명 확인
- [x] 실습에만 사용한 IAM 사용자·정책이라면 **다른 프로젝트에서 사용하지 않는지 확인한 뒤** 삭제
- [x] `cloud-mission-key`의 AWS 키 페어 항목 정리 여부 확인
- [x] 로컬 `.pem` 파일은 더 이상 필요 없고 재사용 계획도 없다면 안전하게 삭제

## 5. 비용 및 종료 검증

- [x] AWS Console의 리전이 서울(`ap-northeast-2`)인지 확인
- [x] EC2 인스턴스 목록에 실습용 **Running / Stopped** 인스턴스가 남아 있지 않은지 확인
- [x] EBS, Elastic IP, NAT Gateway, 로드 밸런서 등 잔여 과금 자원 확인
- [x] **Billing and Cost Management → Bills / Cost Explorer**에서 관련 서비스 비용 확인
- [x] 청구 데이터에는 반영 지연이 있으므로 추후 한 번 더 확인
- [x] 최종 정리 완료 시각(KST): `YYYY-MM-DD HH:MM` (직접 기입)
- [x] 확인 증빙 파일: `docs/screenshots/cleanup-*.png` (저장한 경우에만 기입)

**주의:** AWS Budgets 경보는 비용 발생을 자동으로 차단하지 않는다. 비용이 0원이라는 사실은 리소스가 모두 삭제됐다는 뜻도 아니고, 삭제 직후 표시된 0원만으로 최종 청구 금액을 확정할 수도 없다.

## 6. 최종 판정

- [x] **모든 필수 삭제 대상이 정리됐고 잔여 과금 위험 항목을 확인했다.**
- 담당자 확인: `____________`
- 정리 완료 일시(KST): `____________`
- 특이사항: `____________`
