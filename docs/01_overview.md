# 1. Overview

Fakestack은 OpenStack Fake Service Simulator로서 OpenStack 기반 시스템의 자동화 테스트 및 개발을 지원하기 위해 존재한다.

본 시스템은 실제 OpenStack 환경을 대체하는 것이 아니라,  
OpenStack API와 유사한 인터페이스와 동작을 제공하여 다음을 가능하게 한다:

- 부작용 없는 통합 테스트 수행
- 비동기 상태 전이 재현
- 서비스 간 오케스트레이션 로직 검증
- 오류 시나리오 및 장애 상황 모사
- 점진적 기능 확장

---

# 2. 목적

본 프로젝트의 목적은 다음과 같다:

1. OpenStack API 기반 시스템을 실제 환경 없이 테스트 가능하게 한다.
2. Nova–Neutron–Cinder 을 비롯한 openstack service 간 비동기 오케스트레이션을 재현한다.
3. 상태 전이와 오류 상황을 제어 가능하게 만든다.
4. 향후 이벤트 기반 아키텍처 및 Data Plane 확장이 가능하도록 설계한다.

---

# 3. 적용 범위

## 3.1 초기 대상 서비스

- Keystone
- Nova
- Neutron
- Cinder

## 3.2 점진적 확장

향후 다음과 같은 서비스가 추가될 수 있다:

- Glance
- Placement
- Heat
- 기타 OpenStack 구성 요소

새로운 서비스 또는 새로운 기능 시나리오가 추가될 경우,  
해당 시나리오에 대해 동일한 Phase 1~6 구조를 반복 적용한다.

---

# 4. 시나리오 기반 개발 모델

본 시스템은 **전역 Phase 진행 모델이 아니라, 시나리오 단위 반복 모델**을 따른다.

즉, 새로운 기능 시나리오가 추가될 때마다 다음 단계를 반복한다:

- Phase 1: 성공 경로 최소 구현
- Phase 2: 해당 시나리오의 오류 처리 추가
- Phase 3: 내부 이벤트 구조 확장
- Phase 4: Data Plane 확장
- Phase 5: Fault Injection 적용
- Phase 6: DSL 기반 시나리오 확장

예시:

- Scenario A: Server Create/Delete
- Scenario B: Server Resize
- Scenario C: Volume Snapshot
- Scenario D: Live Migration

각 시나리오는 동일한 성숙도 단계를 거친다.

---

# 5. 대표 초기 시나리오

초기 구현 대상 시나리오:

- 기존 Port 및 Volume을 사용하여 Server 생성
- Server 생성 시 Volume 신규 생성 포함
- Attach / Detach 동작
- Server 삭제
- 비동기 상태 전이 확인

---

# 6. 설계 철학

## 6.1 Determinism 우선

- 모든 비동기 처리는 재현 가능해야 한다.
- 실행 순서는 명확하게 정의된다.
- 테스트 안정성을 성능보다 우선한다.

## 6.2 Control Plane / Data Plane 분리

- Phase 1~3은 Control Plane 중심 구현
- Phase 4에서 선택적으로 Data Plane 확장
- Data Plane은 plugin 형태로 추가된다.

## 6.3 Progressive Expansion

- Phase는 기능 확장 단위이며, 기존 구조를 파괴하지 않는다.
- 새로운 시나리오마다 동일한 단계 구조를 적용한다.

## 6.4 Test-Centric Architecture

- 단일 worker 모델
- 동시성 최소화
- 관측 가능성(Observability) 확보
- Fault Injection 확장 가능 구조

---

# 7. 기술 스택

- Language: Go
- HTTP Framework: Gin
- ORM: GORM
- API Schema: gophercloud 구조체 기반
- Architecture Pattern:
  - DDD
  - Enterprise Application Architecture
- Repository Structure: Monorepo
- 각 서비스는 개별 CLI entrypoint로 실행

---

# 8. 비목표(Non-Goals)

- Production OpenStack 완전 대체
- Hypervisor/KVM 수준 가상화 재현
- 완전한 OpenStack API 100% 구현
- 고성능 분산 메시징 시스템 구현
