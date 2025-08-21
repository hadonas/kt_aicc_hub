# 🏗️ MSA 보드 (Microservices Architecture Board)

> **KT AICC 기반 RAG 상담 지원 서비스의 실시간 마이크로서비스 상태 대시보드**

## 📊 실시간 서비스 상태

### 🟢 정상 | 🟡 경고 | 🔴 장애 | ⚪ 미확인

| 서비스 | 상태 | 응답시간 | CPU | 메모리 | 마지막 체크 |
|--------|------|----------|-----|--------|-------------|
| **QnA Service** | 🟢 | 45ms | 23% | 67% | 2025-01-XX 15:30:25 |
| **RAG Data Service** | 🟢 | 78ms | 18% | 45% | 2025-01-XX 15:30:25 |
| **Frontend** | 🟢 | 32ms | 12% | 34% | 2025-01-XX 15:30:25 |
| **TTS Service** | 🟢 | 156ms | 28% | 52% | 2025-01-XX 15:30:25 |
| **STT Service** | 🟢 | 89ms | 25% | 48% | 2025-01-XX 15:30:25 |
| **API Gateway** | 🟢 | 15ms | 8% | 22% | 2025-01-XX 15:30:25 |

---

## 🏗️ 서비스 아키텍처 맵

```mermaid
graph TB
    subgraph "🌐 클라이언트 레이어"
        U[사용자]
        A[관리자]
    end
    
    subgraph "🚪 API Gateway Layer"
        APIM[Azure API Management<br/>🚦 라우팅 & 인증<br/>📊 모니터링]
    end
    
    subgraph "🎯 핵심 서비스 레이어"
        QNA[QnA Service<br/>🧠 RAG 엔진<br/>🟢 정상<br/>📍 Azure App Service]
        
        RDS[RAG Data Service<br/>📚 데이터 관리<br/>🟢 정상<br/>📍 Azure App Service]
        
        TTS[TTS Service<br/>🎤 음성 합성<br/>🟢 정상<br/>📍 Azure App Service]
        
        STT[STT Service<br/>🎧 음성 인식<br/>🟢 정상<br/>📍 Azure App Service]
    end
    
    subgraph "🖥️ 프론트엔드 레이어"
        FE[Frontend<br/>⚛️ React + TypeScript<br/>🟢 정상<br/>📍 Vercel/Netlify]
    end
    
    subgraph "☁️ 외부 서비스"
        AO[Azure OpenAI<br/>🤖 GPT-4<br/>🔑 API Key]
        
        AS[Azure Speech<br/>🗣️ STT/TTS<br/>🔑 API Key]
        
        MA[MongoDB Atlas<br/>🗄️ 벡터 DB<br/>🔗 연결됨]
    end
    
    subgraph "📊 모니터링 & 로깅"
        AM[Azure Monitor<br/>📈 메트릭 수집]
        
        LA[Log Analytics<br/>📝 로그 분석]
        
        AI[Application Insights<br/>🔍 성능 모니터링]
    end
    
    %% 연결 관계
    U --> APIM
    A --> APIM
    APIM --> QNA
    APIM --> RDS
    APIM --> TTS
    APIM --> STT
    APIM --> FE
    
    QNA --> AO
    QNA --> MA
    RDS --> AO
    RDS --> MA
    TTS --> AS
    STT --> AS
    
    QNA --> AM
    RDS --> AM
    TTS --> AM
    STT --> AM
    AM --> LA
    AM --> AI
    
    %% 상태 표시
    classDef healthy fill:#d4edda,stroke:#155724,stroke-width:2px
    classDef warning fill:#fff3cd,stroke:#856404,stroke-width:2px
    classDef error fill:#f8d7da,stroke:#721c24,stroke-width:2px
    
    class QNA,RDS,TTS,STT,FE healthy
```

---

## 🔗 서비스 의존성 관계

```mermaid
graph LR
    subgraph "📱 사용자 요청 플로우"
        U1[사용자 질문] --> APIM1[API Gateway]
        APIM1 --> STT1[STT Service]
        STT1 --> QNA1[QnA Service]
        QNA1 --> TTS1[TTS Service]
        TTS1 --> U2[음성 답변]
    end
    
    subgraph "📚 문서 관리 플로우"
        A1[관리자] --> APIM2[API Gateway]
        APIM2 --> RDS1[RAG Data Service]
        RDS1 --> QNA2[QnA Service]
    end
    
    subgraph "🔍 데이터 검색 플로우"
        QNA3[QnA Service] --> MA1[MongoDB Atlas]
        MA1 --> QNA4[벡터 검색 결과]
        QNA4 --> AO1[Azure OpenAI]
        AO1 --> QNA5[AI 답변 생성]
    end
```

---

## 📈 성능 메트릭 대시보드

### 🚀 응답 시간 트렌드 (최근 24시간)

```mermaid
graph LR
    subgraph "평균 응답시간"
        QNA_RT[QnA: 45ms<br/>📈 -12%]
        RDS_RT[RAG Data: 78ms<br/>📈 -8%]
        TTS_RT[TTS: 156ms<br/>📈 -5%]
        STT_RT[STT: 89ms<br/>📈 -15%]
    end
    
    subgraph "처리량 (req/min)"
        QNA_TP[QnA: 1,234<br/>📈 +18%]
        RDS_TP[RAG Data: 89<br/>📈 +5%]
        TTS_TP[TTS: 567<br/>📈 +12%]
        STT_TP[STT: 456<br/>📈 +8%]
    end
```

---

## 🚨 알림 및 이벤트

### 📢 최근 알림 (최근 1시간)

| 시간 | 서비스 | 유형 | 메시지 | 상태 |
|------|--------|------|--------|------|
| 15:28:15 | QnA Service | ℹ️ | 정상 상태 확인됨 | ✅ |
| 15:25:42 | TTS Service | ⚠️ | 응답시간 증가 감지 (180ms) | 🔄 |
| 15:22:18 | STT Service | ℹ️ | 정상 상태 확인됨 | ✅ |
| 15:20:05 | RAG Data Service | ℹ️ | 정상 상태 확인됨 | ✅ |

### 🔔 활성 알림

- **없음** - 모든 서비스가 정상 운영 중

---

## 🛠️ 헬스체크 엔드포인트

### 📋 서비스별 상태 확인

```bash
# QnA Service
curl https://your-qna-service.azurewebsites.net/health

# RAG Data Service  
curl https://your-rag-data-service.azurewebsites.net/health

# TTS Service
curl https://your-tts-service.azurewebsites.net/health

# STT Service
curl https://your-stt-service.azurewebsites.net/health

# 통합 헬스체크
curl https://your-api-gateway.azure-api.net/health
```

---

## 📊 리소스 사용량

### 💻 CPU 사용률
- **QnA Service**: 23% (정상 범위: 0-80%)
- **RAG Data Service**: 18% (정상 범위: 0-80%)
- **TTS Service**: 28% (정상 범위: 0-80%)
- **STT Service**: 25% (정상 범위: 0-80%)

### 🧠 메모리 사용률
- **QnA Service**: 67% (정상 범위: 0-85%)
- **RAG Data Service**: 45% (정상 범위: 0-85%)
- **TTS Service**: 52% (정상 범위: 0-85%)
- **STT Service**: 48% (정상 범위: 0-85%)

---

## 🔄 자동화된 복구 작업

### 🚀 현재 실행 중인 작업
- **없음** - 모든 서비스가 정상 상태

### 📋 복구 정책
- **응답시간 > 500ms**: 자동 스케일 아웃
- **CPU > 80%**: 자동 스케일 아웃  
- **메모리 > 85%**: 자동 스케일 아웃
- **연속 3회 실패**: 자동 재시작

---

## 📅 유지보수 일정

### 🗓️ 예정된 작업
- **없음** - 현재 예정된 유지보수 없음

### 📝 최근 작업 내역
- **2025-01-XX**: 시스템 업데이트 완료
- **2025-01-XX**: 성능 최적화 적용
- **2025-01-XX**: 보안 패치 적용

---

## 🔐 보안 상태

### 🛡️ 인증/인가
- **API Gateway**: ✅ 정상
- **Azure Key Vault**: ✅ 정상
- **JWT 토큰**: ✅ 유효

### 🚨 보안 이벤트
- **없음** - 보안 위협 없음

---

## 📱 모바일 대시보드

### 📊 간소화된 상태 보기

```
🟢 QnA Service    🟢 RAG Data
🟢 TTS Service    🟢 STT Service  
🟢 Frontend       🟢 API Gateway

📈 전체 시스템: 정상 운영 중
⏱️  평균 응답시간: 67ms
🚀  처리량: 2,346 req/min
```

---

## 🔄 실시간 업데이트

이 MSA 보드는 **5초마다 자동으로 업데이트**됩니다.

**마지막 업데이트**: 2025-01-XX 15:30:25 UTC

---

## 📞 연락처

### 🆘 장애 발생 시
- **긴급**: 시스템 관리자 (24/7)
- **일반**: 개발팀 (평일 9AM-6PM)

### 📧 알림 설정
- **이메일**: dev-team@company.com
- **Slack**: #system-alerts
- **Teams**: 시스템 모니터링 채널
