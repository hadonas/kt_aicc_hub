# 🎯 마이크로서비스 핵심 이벤트 추출 가이드

> **기능 명세(요구사항/유저 스토리/프로세스 문서)만 보고 도메인 이벤트(과거 시제, 비즈니스 의미 있는 상태 변화)를 추출해, 마이크로서비스 경계와 계약을 설계할 수 있게 한다.**

## 🎯 목표

기능 명세서를 읽고 **도메인 이벤트**를 추출하여:
- 마이크로서비스 경계 설계
- 서비스 간 계약 정의
- 비동기 통신 설계
- 이벤트 기반 아키텍처 구축

---

## 📖 1) 읽는 법: 동사·명사로 분해 → 과거형 이벤트로 재구성

### 🔍 단계별 분석

1. **텍스트에서 동사/행위와 명사/주체를 밑줄 긋는다**
2. **각 행위를 과거 시제로 바꾼다** (도메인 이벤트 규칙)
3. **이벤트의 주체(명사)를 옆에 적는다** (소유자/근본 데이터가 있는 쪽)

### 📝 예시

**원문**: "사용자가 강좌를 구매하면, 결제가 승인되면 수강이 등록되고, 영상 접근 권한이 부여된다."

**분해**:
- 사용자가 강좌를 **구매**하면 → `CoursePurchased` (Course)
- 결제가 **승인**되면 → `PaymentApproved` (Payment)  
- 수강이 **등록**되고 → `EnrollmentCreated` (Enrollment)
- 영상 접근 권한이 **부여**된다 → `LessonAccessGranted` (LessonAccess)

### 📋 룰

**이벤트 이름은 "무엇이(명사) 어떻게 변화했는가(과거형 동사)"로 만든다**

---

## 🔄 2) 상태 전이 찾기: 시작·종료 조건과 게이트 규칙

### 🚀 시작/종료 조건 찾기

**명세서에서 "언제 시작/끝나는가, 선행 조건은 무엇인가" 문장을 표시한다**

- **시작 조건**: "결제 요청이 접수되면 …" → `PaymentRequested`
- **종료 조건**: "승인되면/거절되면 …" → `PaymentApproved` / `PaymentDeclined`
- **게이트(선행) 규칙**: "결제 승인 이전엔 등록 불가" → `EnrollmentCreated`는 `PaymentApproved` 이후에만 발생 가능

### 🔗 게이트 규칙이 곧 경계(서비스) 간의 계약이 된다

---

## 🏗️ 3) 서비스 경계 힌트: 이벤트의 "주어(Owner)"로 나누기

### 📊 같은 명사가 반복되는 이벤트 묶음이 하나의 바운디드 컨텍스트(서비스 경계) 후보다

```
PaymentRequested / PaymentApproved / PaymentRefunded → Payment 서비스
EnrollmentCreated / EnrollmentCanceled → Enrollment 서비스
```

### 🎯 **"최종으로 데이터를 쓴다/변경한다"의 주체가 그 경계의 소유자다** (SOR: System of Record)

---

## ⚡ 4) 비동기 후보·불변식·멱등 키 정하기

### 🔄 동기/비동기 판단

**명세서에서 실시간 동기 여부가 꼭 필요한가를 따진다**

- **반드시 즉시 응답 필요** (OAuth 토큰 발급 등) → **동기 API**
- **약간 지연 허용** (재고 차감, 알림 발송) → **비동기 이벤트**

### 🛡️ 불변식과 멱등 키

**명세 속 핵심 규칙(불변식)을 이벤트 순서/검증으로 고정한다**

- "승인 전 등록 금지"
- "재고 음수 금지"

**멱등 키를 명세에 함께 박아둔다** (중복/지연 안전)

- **결제**: `txId`
- **재고 차감**: `(orderId, sku)`
- **수강 등록**: `(courseId, userId)`

---

## 📋 5) 기능 문장을 → 이벤트로 바꾸는 템플릿(워크시트)

### 📝 입력 문장 예시 (온라인 강의)

> "사용자가 강좌를 구매하면 결제를 승인한다. 승인이 완료되면 수강 등록을 생성하고, 등록이 완료되면 강의 영상 접근 권한을 부여한다. 결제가 거절되면 주문은 취소된다."

### 🎯 추출 이벤트 (과거형)

```
CoursePurchased
PaymentApproved / PaymentDeclined
EnrollmentCreated
LessonAccessGranted
OrderCanceled (보상/실패 흐름)
```

### 🏢 주체(서비스) 매핑

```
Course/Order → Order 서비스
Payment → Payment 서비스
Enrollment → Enrollment 서비스
LessonAccess → Media/Access 서비스
```

### 🔒 게이트 규칙

```
EnrollmentCreated after PaymentApproved
LessonAccessGranted after EnrollmentCreated
```

### 🔑 멱등 키

```
Payment: txId
Enrollment: (courseId, userId)
Access: (userId, courseId)
```

### ⚡ 동기/비동기

```
PaymentRequested → 동기 API(승인 응답 필요)
PaymentApproved → 이벤트 발행(등록 서비스가 구독)
```

### 📄 계약 스케치 (JSON 예시)

```json
{
  "event": "PaymentApproved",
  "version": "v1",
  "txId": "PG-20250901-0001",
  "orderId": "ORD-777",
  "userId": "U-42",
  "approvedAmount": 69000,
  "approvedAt": "2025-09-01T01:02:03Z"
}
```

---

## 📚 6) 요구사항 형태별 추출 요령

### A. 유저 스토리 (As a … I want … so that …)

**"I want"의 동사를 과거형 이벤트로 바꾼다**

**"so that"은 결과/후속 이벤트 또는 게이트 규칙의 단서**

**예시**: "As a student, I want to enroll after payment so that I can watch lessons."

```
PaymentApproved → EnrollmentCreated → LessonAccessGranted
```

### B. 프로세스 다이어그램/BPMN

**상태 변환 노드를 이벤트로, 게이트웨이는 게이트 규칙으로 바꾼다**

### C. 에러/예외 섹션

**실패 케이스는 보상 이벤트로 만든다**

**예시**: "결제 실패 시 주문 취소" → `PaymentDeclined` → `OrderCanceled`

---

## ✅ 7) 체크리스트 (바로 적용)

- [ ] 이벤트 이름이 과거형인가? (명사+과거동사)
- [ ] 주체(Owner)가 분명한가? (누가 최종 변경을 소유?)
- [ ] 게이트 규칙이 문장으로 명문화됐는가?
- [ ] 멱등 키가 정해졌는가?
- [ ] 동기/비동기 분리가 합리적인가? (즉시성 필요만 동기)
- [ ] 실패/보상 이벤트가 정의되어 있는가?
- [ ] v1 스키마 필수 필드가 고정되어 있는가? (버전업 계획 포함)

---

## ❌ 8) 나쁜 패턴 (피하기)

- **명세의 문장을 명령형 이벤트로 옮김**: `DoReduceStock(X)` → `InventoryReduced(O)`
- **모든 걸 동기 체인으로 연결** (피크 시 99th 지연 폭증, 연쇄 장애)
- **latest 하나로만 배포** (재현·롤백 불가)
- **멱등/게이트 규칙을 스키마·코드에 박아두지 않음** (운영 시 규칙 붕괴)

---

## 🚀 9) 빠른 연습 (3분)

### 📝 연습 문제

**기능 문장 2~3개를 고르고 과거형 이벤트로 바꾼다**

**각 이벤트의 주체(서비스)를 적는다**

**선행 조건(게이트) 1~2개를 문장으로 쓴다**

**멱등 키를 붙인다**

**동기/비동기 라벨을 붙인다**

---

## 🎯 KT AICC 프로젝트 적용 예시

### 📋 기능 명세서에서 추출한 이벤트

#### 🧠 QnA Service 도메인
```
QuestionReceived (사용자 질문 수신)
AnswerGenerated (AI 답변 생성)
DocumentRetrieved (관련 문서 검색)
```

#### 📚 RAG Data Service 도메인
```
DocumentUploaded (PDF 문서 업로드)
DocumentProcessed (문서 처리 완료)
VectorEmbedded (벡터 임베딩 생성)
```

#### 🎤 TTS Service 도메인
```
TextReceived (텍스트 수신)
AudioGenerated (음성 파일 생성)
AudioDelivered (음성 파일 전달)
```

#### 🎧 STT Service 도메인
```
AudioReceived (음성 파일 수신)
TextTranscribed (음성 텍스트 변환)
TextDelivered (텍스트 전달)
```

### 🔗 서비스 간 계약

```
AnswerGenerated → AudioGenerated (TTS 서비스가 구독)
TextTranscribed → AnswerGenerated (QnA 서비스가 구독)
DocumentProcessed → AnswerGenerated (QnA 서비스가 구독)
```

### 🔑 멱등 키

```
QuestionReceived: (sessionId, timestamp)
AnswerGenerated: (questionId, modelVersion)
DocumentProcessed: (documentId, chunkIndex)
AudioGenerated: (textHash, voiceType)
```

---

## 📖 참고 자료

- **Domain-Driven Design**: Eric Evans
- **Event Storming**: Alberto Brandolini  
- **Event Sourcing**: Martin Fowler
- **CQRS**: Greg Young

---

## 🎯 다음 단계

1. **이벤트 스토밍 세션** 진행
2. **이벤트 스키마 설계** 
3. **서비스 경계 명확화**
4. **API 계약 정의**
5. **비동기 통신 설계**

---

*이 가이드를 따라하면 기능 명세서만으로도 체계적인 마이크로서비스 아키텍처를 설계할 수 있습니다! 🚀*
