# 🏢 한화손해보험 RAG 기반 상담 지원 서비스

> **한화손해보험의 상품 관련 질문에 대해, 사용자의 자연어 입력을 벡터화하여 사전 임베딩된 답변들과 비교하고, 가장 유사한 답변을 자동으로 추천하는 RAG 기반 상담 지원 서비스**

## 📋 시스템 개요

본 시스템은 **한화손해보험**의 상품 관련 상담을 지원하는 **RAG(Retrieval-Augmented Generation) 기반 AI 상담 시스템**입니다. 사용자의 질문(텍스트/음성)을 받아 관련 보험 상품 정보를 검색하고, AI가 생성한 답변을 음성으로 제공하여 고객 상담 경험을 향상시킵니다.

### 🌟 핵심 특징
- **🤖 RAG 기반 답변 생성**: 벡터 검색을 통한 정확한 보험 정보 제공
- **🎤 음성 상담 지원**: STT(음성→텍스트) + TTS(텍스트→음성) 통합
- **📚 동적 데이터 관리**: 실시간 보험 상품 정보 업데이트
- **🌐 웹 기반 인터페이스**: 직관적인 사용자 경험
- **⚡ 마이크로서비스 아키텍처**: 확장 가능하고 유지보수 용이한 구조

## 🏗️ 시스템 아키텍처

> 💡 **시스템 아키텍처 다이어그램은 draw.io로 작성되어 별도 이미지로 제공됩니다.**

## 🚀 구성 서비스

### 1. **QnA Service** - 핵심 RAG 엔진
- **Repository**: [project03_model](https://github.com/changhyeongHa/project03_model)
- **기능**: 사용자 질문에 대한 AI 답변 생성 및 관련 문서 검색
- **기술**: FastAPI, Azure OpenAI, MongoDB Atlas, LangChain

#### 주요 API
- `POST /qna`: 질문-답변 처리
- `GET /health`: 서비스 상태 확인

### 2. **RAG Data Service** - 데이터 관리
- **Repository**: [rag-data-service](https://github.com/younyoungieo/rag-data-service)
- **기능**: PDF 문서를 처리하여 RAG 시스템을 위한 벡터 데이터베이스 구축
- **기술**: Spring Boot 3.5.4, Java 21, MongoDB, Azure OpenAI
- **배포**: Azure App Service for Containers

#### 주요 API
- `POST /api/v1/documents`: PDF 문서 업로드 및 벡터 처리
- `GET /api/v1/health`: 서비스 상태 확인
- `GET /actuator/health`: Spring Boot Actuator 헬스체크

#### API 특징
- **PDF 처리**: 최대 50MB PDF 지원, 텍스트 추출 및 청크 분할
- **벡터 생성**: Azure OpenAI text-embedding-ada-002 모델 사용
- **자동 처리**: PDF 다운로드 → 텍스트 추출 → 청크 분할 → 벡터 생성 → MongoDB 저장

### 3. **Frontend** - 사용자 인터페이스
- **Repository**: [kt_project_frontend](https://github.com/hadonas/kt_project_frontend)
- **기능**: 웹 기반 상담 인터페이스
- **기술**: React, TypeScript, 모던 UI/UX

### 4. **TTS Service** - 음성 합성
- **Repository**: [tts-service](https://github.com/changhyeongHa/tts-service)
- **기능**: AI 답변을 자연스러운 음성으로 변환
- **기술**: FastAPI, Azure Neural Voice, 다국어 지원

#### 주요 API
- `POST /tts/convert`: 텍스트를 음성으로 변환
- `GET /voices`: 사용 가능한 음성 목록
- `GET /health`: 서비스 상태 확인

### 5. **STT Service** - 음성 인식
- **Repository**: [stt-service](https://github.com/changhyeongHa/stt-service)
- **기능**: 사용자 음성 질문을 텍스트로 변환
- **기술**: FastAPI, Azure Cognitive Services, 다국어 지원

#### 주요 API
- `POST /stt/convert`: 오디오를 텍스트로 변환
- `GET /health`: 서비스 상태 확인

## 🛠️ 기술 스택

| 구분 | 기술 | 용도 |
|------|------|------|
| **API Gateway** | Azure API Management | API 라우팅, 인증, 모니터링 |
| **Backend Framework** | FastAPI, Spring Boot | 마이크로서비스 API 서버 |
| **AI/ML** | Azure OpenAI (GPT-4, text-embedding-ada-002) | 자연어 처리, 답변 생성, 벡터 임베딩 |
| **Vector Database** | MongoDB Atlas | 벡터 검색 및 문서 저장 |
| **Speech Services** | Azure Cognitive Services | STT/TTS 처리 |
| **Frontend** | React + TypeScript | 사용자 인터페이스 |
| **Container** | Docker | 서비스 컨테이너화 |
| **Language** | Python 3.11+, Java 21 | 백엔드 서비스 |
| **Runtime** | Uvicorn, JVM | ASGI 웹 서버, Java 런타임 |

## 📡 API 통합 예시

### 🔄 전체 상담 플로우

#### 1. 음성 상담 플로우
```mermaid
sequenceDiagram
    participant U as User
    participant F as Frontend
    participant APIM as API Management
    participant STT as STT Service
    participant QnA as QnA Service
    participant AO as Azure OpenAI
    participant M as MongoDB Atlas
    participant TTS as TTS Service
    
    U->>F: 음성으로 질문
    F->>APIM: 오디오 파일 전송
    APIM->>STT: STT 서비스로 라우팅
    STT->>QnA: 변환된 텍스트 전달
    QnA->>M: 벡터 유사도 검색 요청
    M->>QnA: 관련 문서 청크 반환
    QnA->>AO: GPT-4 모델로 답변 생성 요청
    AO->>QnA: AI 답변 반환
    QnA->>TTS: AI 답변을 TTS로 직접 전달
    TTS->>APIM: 음성 파일 반환
    APIM->>F: 음성 파일 전달
    F->>U: 음성 답변 재생
```

#### 2. 텍스트 상담 플로우
```mermaid
sequenceDiagram
    participant U as User
    participant F as Frontend
    participant APIM as API Management
    participant QnA as QnA Service
    participant AO as Azure OpenAI
    participant M as MongoDB Atlas
    
    U->>F: 텍스트로 질문 입력
    F->>APIM: 텍스트 질문 전송
    APIM->>QnA: QnA 서비스로 라우팅
    QnA->>M: 벡터 유사도 검색 요청
    M->>QnA: 관련 문서 청크 반환
    QnA->>AO: GPT-4 모델로 답변 생성 요청
    AO->>QnA: AI 답변 반환
    QnA->>APIM: AI 답변 + 관련 문서
    APIM->>F: 답변 전달
    F->>U: 텍스트 답변 표시
```

#### 3. 문서 추가 워크플로우
```mermaid
sequenceDiagram
    participant A as Admin
    participant F as Frontend
    participant APIM as API Management
    participant RDS as RAG Data Service
    participant AO as Azure OpenAI
    participant M as MongoDB Atlas
    participant QS as QnA Service
    
    A->>F: PDF 문서 URL + 메타데이터 입력
    F->>APIM: 문서 추가 요청 전송
    APIM->>RDS: RAG Data Service로 라우팅
    RDS->>RDS: PDF 다운로드 및 텍스트 추출
    RDS->>RDS: 텍스트 청크 분할
    RDS->>AO: 각 청크의 벡터 임베딩 요청
    AO->>RDS: 1536차원 벡터 반환
    RDS->>M: 청크 + 벡터 + 메타데이터 저장
    RDS->>APIM: 처리 완료 응답
    APIM->>F: 결과 전달
    F->>A: 처리 완료 알림
    Note over QS: 이제 QnA Service에서<br/>새로운 문서 정보 검색 가능
```

### 📝 API 호출 예시

#### 1. 음성 상담 처리
```bash
# 1. STT: 음성을 텍스트로 변환
curl -X POST "https://your-stt-service.azurewebsites.net/stt/convert" \
  -F "audio_file=@question.wav" \
  -F "locale=ko-KR"

# 2. QnA: AI 답변 생성
curl -X POST "https://your-qna-service.azurewebsites.net/qna" \
  -H "Content-Type: application/json" \
  -d '{"input_message": "자동차보험료 계산 방법 알려줘"}'

# 3. TTS: 답변을 음성으로 변환
curl -X POST "https://your-tts-service.azurewebsites.net/tts/convert" \
  -H "Content-Type: application/json" \
  -d '{"text": "자동차보험료는 다음과 같이 계산됩니다..."}'
```

#### 2. 텍스트 상담 처리
```bash
# QnA: 텍스트 질문에 대한 AI 답변 생성
curl -X POST "https://your-qna-service.azurewebsites.net/qna" \
  -H "Content-Type: application/json" \
  -d '{"input_message": "수렵보험의 보장 범위는 어떻게 되나요?"}'
```

#### 3. 새로운 보험 상품 문서 추가
```bash
# RAG Data Service: PDF 문서 업로드 및 벡터 처리
curl -X POST "https://your-rag-data-service.azurewebsites.net/api/v1/documents" \
  -H "Content-Type: application/json" \
  -d '{
    "link": "https://www.hwgeneralins.com/upload/hmpag_upload/product/hunt(2506)_03.pdf",
    "product_group": "일반보험-종합",
    "product_name": "수렵보험",
    "sale_period": "2025.06.30~현재",
    "document_type": "약관"
  }'
```

## 🚀 설치 및 실행

### 사전 요구사항
- Azure 구독 (OpenAI, Speech Services)
- MongoDB Atlas 계정

### 1. 저장소 클론
```bash
# 메인 시스템 저장소
git clone https://github.com/your-org/kt_aicc_hub.git
cd kt_aicc_hub

# 각 서비스 저장소
git clone https://github.com/changhyeongHa/project03_model.git qna-service
git clone https://github.com/younyoungieo/rag-data-service.git rag-data-service
git clone https://github.com/hadonas/kt_project_frontend.git frontend
git clone https://github.com/changhyeongHa/tts-service.git tts-service
git clone https://github.com/changhyeongHa/stt-service.git stt-service
```

### 2. 환경 변수 설정
각 서비스의 환경 변수는 Azure App Service 설정에서 구성됩니다.

**필수 환경 변수:**
- `AZURE_OPENAI_API_KEY`: Azure OpenAI API 키
- `AZURE_OPENAI_ENDPOINT`: Azure OpenAI 엔드포인트
- `AZURE_OPENAI_DEPLOYMENT_NAME`: Azure OpenAI 배포명 (예: text-embedding-ada-002)
- `AZURE_OPENAI_API_VERSION`: Azure OpenAI API 버전 (예: 2023-05-15)
- `AZURE_SPEECH_KEY`: Azure Speech Service API 키
- `AZURE_SPEECH_REGION`: Azure Speech Service 지역
- `MONGODB_URI`: MongoDB Atlas 연결 문자열
- `MONGODB_DB`: MongoDB 데이터베이스명 (예: insurance)
- `MONGODB_COLLECTION`: MongoDB 컬렉션명 (예: documents)

## 📊 시스템 모니터링

### 헬스체크
```bash
# 각 서비스 상태 확인
curl https://your-qna-service.azurewebsites.net/health      # QnA Service
curl https://your-rag-data-service.azurewebsites.net/health # RAG Data Service
curl https://your-stt-service.azurewebsites.net/health     # STT Service
curl https://your-tts-service.azurewebsites.net/health     # TTS Service
```

## 📚 추가 문서

각 서비스의 상세한 사용법과 API 명세는 다음 문서를 참조하세요:

- **QnA Service**: [project03_model](https://github.com/changhyeongHa/project03_model)
- **RAG Data Service**: [rag-data-service](https://github.com/younyoungieo/rag-data-service)
- **Frontend**: [kt_project_frontend](https://github.com/hadonas/kt_project_frontend)
