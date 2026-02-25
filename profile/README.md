

## 프로젝트 소개
**Courm**은 대규모 트래픽 환경에서도 중단 없는 서비스와 데이터 무결성을 보장하는 **고가용성 MSA 이커머스 플랫폼**입니다. <br />
단순한 기능 구현을 넘어, **Transactional Outbox Pattern**과 **Saga Pattern**을 통해 마이크로서비스 간의 강력한 데이터 정합성을 확보했습니다. <br />
Terraform을 이용한 **IaC(Infrastructure as Code)**와 ArgoCD 기반의 **GitOps** 환경을 구축하여 인프라부터 서비스까지 선언적으로 관리합니다.


## 팀원 소개

| Project Leader CICD | Backend | Infra | DevOps |
| :------------------: | :--------------------: | :--------------------: | :-----------------: |
| <img src="https://avatars.githubusercontent.com/boxty123" width="100"/> | <img src="https://avatars.githubusercontent.com/mingdul" width="100"/> | <img src="https://avatars.githubusercontent.com/seoyun0311" width="100"/> | <img src="https://avatars.githubusercontent.com/yyytgf123" width="100"/> |
| [@boxty123](https://github.com/boxty123) | [@mingdul](https://github.com/mingdul) | [@seoyun0311](https://github.com/seoyun0311) | [@yyytgf123](https://github.com/yyytgf123) |
| 이정현 | 장지민 | 양서윤 | 이현호 |

## 핵심 기능

- **Transactional Outbox & Saga Pattern** DB 트랜잭션과 메시지 발행의 원자성을 보장하여 서비스 간 데이터 불일치를 원천 차단하고 비동기 기반의 안정적인 주문 흐름을 구현했습니다.

- **GitOps 기반 배포 자동화 (App of Apps)** `courm-bootstrap` 레포지토리를 통해 인프라, 플랫폼, 마이크로서비스를 ArgoCD로 통합 관리하며 선언적 배포 환경을 구축했습니다.

- **Blue/Green 무중단 배포 전략** Argo Rollouts를 활용하여 신규 버전 배포 시 리스크를 최소화하고, 안정적인 트래픽 전환 및 롤백 프로세스를 제공합니다.

- **동적 노드 오토스케일링 (Karpenter)** 워크로드 기반의 실시간 노드 프로비저닝을 통해 자원 효율성을 극대화하고 트래픽 급증에 유연하게 대응합니다.

- **Full-Stack 관측성 (LGTM Stack)** Loki, Grafana, Tempo, Prometheus를 연동하여 분산 환경에서의 로그 추적, 메트릭 모니터링, 트레이싱을 통합 관리합니다.

## 기술 스택

| 영역 | 사용 기술 |
|------|-----------|
| **Backend** | Java 17, Spring Boot 3.x, Spring Data JPA, Querydsl, Spring Security, Gradle, JUnit5 |
| **Database** | PostgreSQL 15, Redis, Amazon RDS, Amazon ElastiCache |
| **Message Broker** | Apache Kafka |
| **Infra** | AWS (EKS, VPC, ALB, NAT Gateway, S3), Terraform, Kubernetes, Helm, Istio, Karpenter |
| **DevOps** | ArgoCD, Argo Rollouts, Jenkins|
| **Monitoring** | Prometheus, Grafana, Loki, Tempo (LGTM Stack), OpenTelemetry |

## 아키텍처

<details>
<summary> 아키텍처 상세 보기 </summary>
<div align="center">
<img width="914" height="632" alt="Image" src="https://github.com/user-attachments/assets/a75c09c6-5836-4a5c-b3e0-dabc172f104a" />
</div>
</details>

### 🏗 인프라 구조 (EKS & Network)
* **VPC Isolation**: Public/App/Data/MQ/Mgmt Subnet 분리를 통한 보안 강화
* **Traffic Path**: Internet → ALB → Istio Ingress Gateway → VirtualService → Service Pod
* **Node Strategy**: 역할별 Node Group 분리 (Management, Service, Data, Monitoring, CI)

### 🔄 데이터/이벤트 흐름 (Transactional Outbox)
1. **API Request**: 주문 생성 요청
2. **Local Transaction**: 비즈니스 데이터(Order) + 이벤트(Outbox) 동일 DB 커밋
3. **Event Publish**: Outbox Publisher가 Kafka(`order-events`)로 메시지 발행
4. **Event Consume**: 결제/상품 서비스가 메시지 수신 후 로직 수행 및 보상 트랜잭션 처리
