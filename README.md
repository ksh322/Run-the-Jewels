# [Project] B2B Jewel-Mall : B2B 귀금속 수출업체 소개 랜딩 페이지 & 고객, order 관리 시스템

## 1. 프로젝트 개요

-   **목적**: 귀금속 수출 산업의 디지털 전환(DX)을 위한 주문 및 인보이스 관리 시스템 구축.
    
-   **배경**:  **AS-IS**:현업의 수동 고객 오더 메신저로 받기 및 엑셀 인보이스 작업  **TO-BE**: 자동화, API 기반의 체계적인 장바구니 , 고객정보 데이터 관리 프로세스 수립.
    
-   **기대효과**: 실제 상용 서비스 수준의 ERD/API 설계 및 배포 경험 확보.
    

----------

## 2. 개발 요구사항 명세 (MVP)

### 🛠 핵심 기능 (Core Features)

1.  **인증 및 인가 (Auth)**
    
    -   Spring Security 기반 보안 설정.
        
    -   바이어 접근성 향상을 위한 Google/소셜 로그인 연동.
        
2.  **회원 및 업체 관리 (CRM)**
    
    -   B2B 기업 회원 등급(Tier) 및 바이어 정보 관리.
        
3.  **주문 관리 시스템 (Core OMS)**
    
    -   수동 엑셀 인보이스를 대체하는 디지털 주문서 자동 생성.
        
    -   주문 생명주기 관리: `접수` → `결제 확인` → `배송` → `완료`.
        
4.  **장바구니 및 견적함**
    
    -   상품 담기 기능.
        

### 💻 기술 스택 (Tech Stack)

-   **Backend**: Java/Kotlin + SpringBoot (현업 표준 준수)
    
-   **Frontend**: JavaScript React (컴포넌트 기반 UI)
    
-   **Database**: MySQL PostgreSQL 중 택 1 (ERD 기반 설계)
    
-   **Infrastructure**: AWS (Free-tier), Docker (컨테이너화)
    
-   **Collaborator Tools**: Figma, GitHub, Slack, Trello
    

----------

## 3. 팀 구성 및 R&R (Role & Responsibilities)

**이름**

**역할**

**주요 담당 업무**

**17 김상호님**

**기획 / Infra /QA**

비즈니스 로직 정의, 요구사항 명세 작성, 인프라 보조 및 비용 관리

**진욱님**

**Frontend**

Figma 와이어프레임 구현, React 클라이언트 개발, API 연동

**상우님**

**Backend / Infra**

ERD 설계 및 DB 구축, SpringBoot API 개발, CI/CD 파이프라인

**현직 멘토**

**Advisor**
현대오토에버 현직자
기술 스택 의사결정 지원 및 인프라 구축 가이드

----------

## 4. 8주 단기 로드맵 (Roadmap)

**주차**

**단계**

**주요 과업 (Milestones)**

**1-2주**

**기획 및 설계**

오픈소스(Saleor 등) 벤치마킹, ERD 설계, API 명세서 확정, Figma 시안

**3-4주**

**집중 개발**

Auth(로그인) 기능, 장바구니/주문 생성 로직 개발, FE-BE API 연동

**5-6주**

**인프라 및 배포**

AWS 환경 구축, Docker 컨테이너화, 도메인 연결 및 통합 테스트

**마무리**

QA 및 버그 수정, 프로젝트 문서화(README/Claude.md)

----------

## 5. 협업 및 관리 가이드

-   **Documentation**: 핵심 비즈니스 로직(언어 전환, 동시성 제어, 세션 만료 등) 및 API 엔드포인트는 `Claude.md`에 상시 업데이트.
    
-   **Communication**: 회의 및 정보공유 Discord 모든 의사결정 Slack에 기록하고, 작업 진행도는 Trello 칸반 보드로 시각화.
    
-   **Code Quality**: 현직자 멘토의 기술 자문 거쳐 Main Branch Merge.
