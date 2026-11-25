<p align="right"><strong>KR</strong> | <a href="README.en.md">EN</a></p>

# Dver·Dva

**SSAFY 15기 구미 D109 · 디버(Dver) 기업연계 프로젝트**

![Dver·Dva 프로젝트 표지](assets/d109-ppt-cover.png)

## 📌 목차

- [📦 DVA](#-dva)
  - [서비스 흐름](#-dva-서비스-흐름)
  - [시연 영상](#-시연-영상)
  - [관리자 웹](#-관리자-웹)
  - [AI 상담 기술](#-ai-상담-기술)
  - [아키텍처](#-아키텍처)
- [🎙 VOMI](#-vomi)
  - [통화 화면과 시연](#-통화-화면과-시연)
  - [서비스 흐름](#-vomi-서비스-흐름)
  - [음성 합성](#-음성-합성)
- [🛠 기술 스택](#-기술-스택)
- [📁 저장소 구조](#-저장소-구조)
- [👥 팀원](#-팀원)

## 📦 DVA

**전화로 택배를 접수하는 AI 상담원과 관리자 웹**

고객이 전화로 배송 정보를 말하면 AI가 주소와 물품을 확인합니다. 주문은 결제 조건에 맞춰 디버에 접수되며, 운영자는 웹에서 통화와 주문 내역을 확인합니다.

### 🔄 DVA 서비스 흐름

![DVA 서비스 흐름. 디버 PBX가 전화를 연결하면 AI가 발신번호, 주소, 배송 정보를 확인하고 주문을 백엔드로 전달합니다. 개인 카드 결제 또는 기업 월결제·부서 그룹결제를 확인한 뒤 디버에 접수합니다. 통화 중단 시 수집한 정보를 미완료 주문으로 남기고 S3 녹취는 상담 기록에 연결합니다.](assets/dva-service-flow.png)

### 🎬 시연 영상

**서비스 소개 · 54초**

https://github.com/user-attachments/assets/ec853ec0-89d9-4a39-928c-7d1c80c8dfd4

### 🖥 관리자 웹

관리자 웹을 모의 데이터로 실행해 캡처한 화면입니다.

| 홈 | 주문 |
| :---: | :---: |
| <img src="assets/dva-admin-home.jpg" width="390" alt="미완료 주문과 미답변 문의를 보여주는 관리자 홈"> | <img src="assets/dva-admin-orders.jpg" width="390" alt="검색과 상태 필터가 있는 주문 화면"> |
| 처리할 일과 최근 활동 | 주문 검색과 접수 상태 |

| 통화 | AI 상담가 | 통계 |
| :---: | :---: | :---: |
| <img src="assets/dva-admin-calls.jpg" width="260" alt="통화 이력과 분석 상태 필터"> | <img src="assets/dva-admin-personas.jpg" width="260" alt="AI 상담가의 목소리 선택, 수정 기록, 프롬프트"> | <img src="assets/dva-admin-stats.jpg" width="260" alt="접수 소요 시간과 응답 대기 통계"> |
| 통화 이력과 녹취 | 목소리·버전·프롬프트 | 접수 시간과 응답 대기 |

- **시나리오 시험** · AI 상담가 프롬프트를 통화에 반영하기 전, 가상 고객과 시나리오 10개를 음성으로 통화해 자동 판정합니다.
- **통화 불만 판정** · 통화가 끝나면 고객 불만과 AI 응대 오류 5유형을 판정해 통화 상세와 통계에 보여 줍니다(고객 불만 정밀도 96%).

### 🤖 AI 상담 기술

![GPT-Live-1 음성 대화, Decisions API 판단, Responses Delegation 정보 추출 흐름](assets/dva-ai-flow.png)

| 기술 | 역할 |
| --- | --- |
| **GPT-Live-1** | PBX로 들어오는 통화에서 양방향 음성 응대와 끼어들기 처리 |
| **Decisions API · `gpt-6-luna`** | 접수 의도, 확인·정정, 주소 후보 선택 판단 |
| **Responses Delegation** | 이름·전화번호·주소·물품 추출과 업무 도구 호출 |
| **FAQ 검색 · RAG** | Qdrant 임베딩과 Kiwi·BM25 검색을 RRF로 합친 뒤 Qwen3로 재정렬 |
| **한국어 주소 보정** | 도로명 주소 DB와 자모 단위 편집 거리로 주소 후보 검색 |

- **판단 응답** · Decisions API 웜 호출 중앙값 **251ms**, P95 **284ms** (53회)
- **FAQ 검색** · Qwen3 재정렬 적용 후 Top 1 **71% → 96%**, Top 3 **86% → 100%**

### 📊 RAG 성능 및 응답 품질 평가

직접 구축·검수한 **95문항 골든 데이터셋**을 기반으로 검색 정확도와 실제 AI 상담 응답 품질을 평가했습니다.

#### 검색 성능

| 평가 지표 | 초기 | 최종 |
| --- | ---: | ---: |
| Hit@1 | 70.5% (67/95) | **95.8% (91/95)** |
| Hit@3 | 86.3% (82/95) | **100% (95/95)** |

- **평균 검색 지연** · 270ms
- **P95 검색 지연** · 328ms
- **검색 파이프라인** · Dense + Kiwi/BM25 → RRF → Qwen3 Reranker → Top3

#### Ragas 기반 응답 품질 평가

검색 결과뿐 아니라 **실제 Live 음성 답변의 품질**을 검증하기 위해 Ragas 평가를 진행했습니다.

| 평가 지표 | 텍스트 LLM | Live 음성 답변 |
| --- | ---: | ---: |
| Faithfulness | 0.9612 | 0.8579 |
| Answer Relevancy | 0.5934 | 0.4825 |
| Context Precision | 0.9807 | 0.9895 |
| Context Recall | 0.9789 | 0.9895 |
| Factual Correctness F1 | - | 0.7214 |


### 🏗 아키텍처

아래에서 구조를 선택하면 한 번에 한 장씩 볼 수 있습니다.

<details name="dva-architecture" open>
<summary><strong>모듈러 모놀리스 · AWS Lightsail</strong></summary>

![AWS Lightsail의 Nginx, NestJS 모듈, 공유 PostgreSQL, S3와 디버 주문 API 구성](assets/dva-monolith.png)

Docker Compose에서 NestJS 모듈과 PostgreSQL을 함께 운영합니다. 관리자 웹과 AI 에이전트는 Nginx를 통해 업무 API에 접근합니다.

</details>

<details name="dva-architecture">
<summary><strong>MSA 검증 환경 · EC2와 k3s</strong></summary>

![EC2와 k3s의 Traefik, 다섯 NestJS 서비스와 서비스별 DB, Outbox·Inbox 구성](assets/dva-msa.png)

업무를 다섯 서비스와 개별 DB로 분리했습니다. k3s·Traefik으로 배포하고 Outbox·Inbox로 이벤트 재전송과 중복을 처리했습니다.

</details>

[모놀리스 ERD · DBML](docs/database/monolith.dbml)

## 🎙 VOMI

![VOMI 프로젝트 표지](assets/vomi-ppt-cover.png)

**말하기 어려운 순간, 입력한 답변을 내 목소리로 전하는 통화 앱**

조용한 곳에서 전화를 받으면 추천 답변을 고르거나 직접 입력할 수 있습니다. 상대방은 텍스트를 바로 받고, 등록한 목소리로 합성한 답변도 듣습니다.

### 📱 통화 화면과 시연

<img src="assets/vomi-call.jpg" width="235" alt="VOMI 통화 화면의 대화 내역, 추천 답변, 직접 입력">

**음성 합성 비교 · 13초**

https://github.com/user-attachments/assets/fae885d6-8131-4b5e-bf38-7e27e3afc403

**같은 문장 듣기** · [원본 녹음](assets/vomi-original.wav) · [ElevenLabs](assets/vomi-elevenlabs.wav) · [OmniVoice + LoRA](assets/vomi-omnivoice.wav)

### 🔄 VOMI 서비스 흐름

![VOMI 서비스 흐름. 목소리를 등록하고 통화 상대가 수락하면 두 앱이 같은 LiveKit 방에 접속합니다. 답변을 고르거나 입력하면 텍스트는 즉시 표시되고, NestJS가 등록 목소리로 합성한 PCM 청크는 LiveKit 바이트 스트림을 통해 상대 앱에서 순서대로 재생됩니다.](assets/vomi-service-flow.png)

### 🔊 음성 합성

처음에는 **ElevenLabs**로 통화 중 음성 합성을 구현했습니다. 실시간 통화에서 기대한 성능을 얻기 어려워 로컬에서 실행하는 **OmniVoice**를 **LoRA**로 미세조정했습니다.

- **학습 데이터** · 부산 말투로 녹음한 2분 33초 분량을 53개 클립으로 나눠 사용했습니다.
- **합성 속도** · FlashInfer 적용 후 RTX 4070에서 50자 문장 합성 시간 중앙값이 **784ms → 412ms**로 줄었습니다.

## 🛠 기술 스택

| 영역 | DVA | VOMI |
| --- | --- | --- |
| 화면 | React · TypeScript | Kotlin · Jetpack Compose |
| 서버·DB | NestJS · FastAPI · PostgreSQL | NestJS · PostgreSQL |
| 음성·AI | GPT-Live-1 · Decisions API · Qdrant · Qwen3 | LiveKit · ElevenLabs · Gemini Nano · OmniVoice |
| 배포·연동 | AWS Lightsail · S3 · Docker Compose · Jenkins · k3s | LiveKit Cloud · Firebase Cloud Messaging · Docker Compose |

## 📁 저장소 구조

```text
.
├── frontend/            # DVA 관리자 웹
├── ai/                  # DVA 실시간 음성 상담 · FAQ 검색
├── backend/             # DVA NestJS 모듈러 모놀리스
├── msa/                 # DVA MSA 검증 환경
│   ├── services/        # 운영 · 고객 · 상담 · 주문 · 배송 서비스
│   ├── liquibase/       # 서비스별 DB 마이그레이션
│   └── deploy/          # k3s 배포 설정
├── kick/                # VOMI
│   ├── vomi/            # Android 앱
│   ├── backend/         # 통화 · 친구 · 음성 합성 API
│   └── ai/              # 온디바이스 AI PoC
├── infra/               # AWS · Nginx · 배포 · 관측 설정
├── postman/             # 외부 API 시뮬레이션
├── docs/                # 설계 · API 문서
├── exec/                # 프로젝트 포팅 매뉴얼
├── assets/              # README 이미지 · 영상 · 음성
├── compose.yaml         # 로컬 실행 환경
└── compose.prod.yaml    # 운영 실행 환경
```

## 👥 팀원

| [<img src="https://github.com/jlkim909.png?size=160" width="80" alt="유민국">](https://github.com/jlkim909) | [<img src="https://github.com/Jiwoo-Kim0.png?size=160" width="80" alt="김지우">](https://github.com/Jiwoo-Kim0) | [<img src="https://github.com/jihwan4155.png?size=160" width="80" alt="신지환">](https://github.com/jihwan4155) | [<img src="https://github.com/hantawon123.png?size=160" width="80" alt="한태원">](https://github.com/hantawon123) | [<img src="https://github.com/sejhyun13.png?size=160" width="80" alt="현승준">](https://github.com/sejhyun13) | [<img src="https://github.com/42JH.png?size=160" width="80" alt="이재훈">](https://github.com/42JH) |
| :---: | :---: | :---: | :---: | :---: | :---: |
| [유민국](https://github.com/jlkim909) | [김지우](https://github.com/Jiwoo-Kim0) | [신지환](https://github.com/jihwan4155) | [한태원](https://github.com/hantawon123) | [현승준](https://github.com/sejhyun13) | [이재훈](https://github.com/42JH) |
| 팀장 · 백엔드<br>주문·결제 | 관리자 웹<br>상담가 설정 | DVA 전화 상담<br>RAG 설계/평가<br>VOMI 온디바이스 AI| 백엔드·MSA·CI<br>VOMI 통화·음성 | DVA 대화 제어·Decision API<br>주소·모델 위임 | 프론트엔드 리드·UI 디자인·배포<br>시나리오 시험·불만 판정 |
