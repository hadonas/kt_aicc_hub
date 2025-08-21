# 📋 API 계약 명세
# KT AICC 기반 RAG 상담 지원 서비스

> **KT의 AICC(AI Contact Center)를 본떠 만든 RAG(Retrieval-Augmented Generation) 기반 AI 상담 지원 서비스의 API 계약 명세입니다. 모든 서비스의 Request/Response DTO와 상세한 API 정보를 포함합니다.**

## QnA Service API 계약

### 서비스 개요
- **서비스명**: QnA Service
- **설명**: RAG 기반 AI 답변 생성 및 관련 문서 검색 서비스
- **기술**: FastAPI, Azure OpenAI, MongoDB Atlas, LangChain

### API 엔드포인트

**1. GET / - 헬스체크용 루트 엔드포인트**

**Response (200 OK):**
```json
{
    "message": "AI Q&A Service is running",
    "status": "healthy"
}
```

**2. GET /health - 상세 헬스체크 엔드포인트**

**Response (200 OK):**
```json
{
    "status": "healthy",
    "timestamp": "2024-01-01T00:00:00Z",
    "service": "AI Q&A Service",
    "version": "1.0.0",
    "message": "All required environment variables are set"
}
```

**3. POST /qna - 질문-답변 처리**

**Request DTO:**
```json
{
    "input_message": "자동차보험료 계산 방법 알려줘"
}
```

**Response (200 OK):**
```json
{
    "success": true,
    "messages": [
        {"HumanMessage": "자동차보험료 계산 방법 알려줘"},
        {"AIMessage": "자동차보험료는 다음과 같이 계산됩니다..."}
    ],
    "citations": [
        {"title": "보험료계산서.pdf", "page": 15},
        {"title": "자동차보험가이드.pdf", "page": 23}
    ]
}
```

**Response (422/500 Error):**
```json
{
    "success": false,
    "error": "Error message"
}
```

## RAG Data Service API 계약

### 서비스 개요
- **서비스명**: RAG Data Service
- **설명**: PDF 문서를 처리하여 RAG 시스템을 위한 벡터 데이터베이스를 구축하는 서비스

### API 엔드포인트

**1. POST /api/v1/documents - 문서 업로드 및 처리**

**Request DTO:**
```json
{
  "link": "https://example.com/document.pdf",
  "product_group": "일반보험-종합",
  "product_name": "수렵보험",
  "sale_period": "2025.06.30~현재",
  "document_type": "약관"
}
```

**필드 설명:**
| 필드명 | 타입 | 필수 | 설명 | 예시 |
|--------|------|------|------|------|
| link | URL | ✅ | PDF 문서의 다운로드 URL | `"https://www.hwgeneralins.com/upload/..."` |
| product_group | String | ✅ | 상품의 그룹 분류 | `"일반보험-종합"`, `"자동차보험"` |
| product_name | String | ✅ | 상품의 구체적인 이름 | `"수렵보험"`, `"자동차종합보험"` |
| sale_period | String | ✅ | 상품의 판매 기간 | `"2025.06.30~현재"` |
| document_type | String | ✅ | 문서의 유형 | `"약관"`, `"설명서"`, `"안내서"` |

**Success Response (200 OK):**
```json
{
  "status": "success",
  "message": "문서가 성공적으로 처리되었습니다. 353개의 청크가 생성되었습니다.",
  "documentId": "ce5b3485-dcac-4351-a3f3-0b652c337ed2"
}
```

**Error Response (500 Internal Server Error):**
```json
{
  "status": "error",
  "message": "PDF 처리 중 오류가 발생했습니다: [에러 상세 내용]",
  "documentId": "ce5b3485-dcac-4351-a3f3-0b652c337ed2"
}
```

**2. GET /api/v1/health - 서비스 상태 확인**

**Response (200 OK):**
```json
{
  "status": "healthy",
  "message": "서비스가 정상적으로 실행 중입니다",
  "documentId": null
}
```

**3. GET /actuator/health - Spring Boot Actuator 헬스체크**

**Response (200 OK):**
```json
{
  "status": "UP",
  "components": {
    "diskSpace": {
      "status": "UP",
      "details": {
        "total": 36670308352,
        "free": 13235208192,
        "threshold": 10485760,
        "path": "/app/.",
        "exists": true
      }
    },
    "mongo": {
      "status": "UP",
      "details": {
        "maxWireVersion": 25
      }
    },
    "ping": {
      "status": "UP"
    },
    "ssl": {
      "status": "UP",
      "details": {
        "validChains": [],
        "invalidChains": []
      }
    }
  }
}
```



## TTS Service API 계약

### 서비스 개요
- **서비스명**: TTS Service
- **설명**: 텍스트를 자연스러운 음성으로 변환하는 서비스
- **기술**: Azure Neural Voice, FastAPI

### API 엔드포인트

**1. GET / - 서비스 루트 엔드포인트**
**기본 상태 확인**

**Response (200 OK):**
```json
{
  "message": "TTS Service is running",
  "status": "healthy"
}
```

**2. GET /health - 서비스 상태 확인**
**Azure Speech 연결 테스트 포함**

**Response (200 OK):**
```json
{
  "status": "healthy",
  "service": "TTS Service", 
  "version": "1.0.0",
  "azure_speech_region": "koreacentral",
  "default_voice": "ko-KR-SunHiNeural"
}
```

**3. POST /tts/convert - 텍스트를 음성 파일로 변환**
**WAV 파일 직접 반환**

**Request DTO:**
```json
{
  "text": "변환할 텍스트 내용",
  "voice_name": "ko-KR-SunHiNeural"
}
```

**요청 필드 설명:**
| 필드 | 타입 | 필수 | 기본값 | 설명 |
|------|------|------|--------|------|
| text | string | ✅ | - | 변환할 텍스트 (최대 1000자) |
| voice_name | string | ❌ | ko-KR-SunHiNeural | 음성 종류 |

**Response (200 OK) - WAV 파일:**
```
HTTP/1.1 200 OK
Content-Type: audio/wav
Content-Disposition: attachment; filename="tts_output.wav"
X-Voice-Name: ko-KR-SunHiNeural
X-Text-Length: 50
X-TTS-Success: true
X-TTS-Message: 음성 합성 완료

[WAV 오디오 파일 바이너리 데이터]
```

**4. POST /tts/convert-json - JSON 응답 형태**
**텍스트를 음성으로 변환**

**Request DTO:**
```json
{
  "text": "변환할 텍스트 내용",
  "voice_name": "ko-KR-SunHiNeural"
}
```

**Response (200 OK) - JSON:**
```json
{
  "success": true,
  "message": "음성 합성 완료",
  "filename": "/tmp/temp_audio_file.wav",
  "voice_name": "ko-KR-SunHiNeural",
  "text_length": 25
}
```

**5. POST /tts/convert-rag-response - RAG 응답을 Multipart 형태로 변환**
**JSON + WAV 동시 반환**

**Request DTO:**
```json
{
  "success": true,
  "messages": [
    {"HumanMessage": "자동차보험료 계산 방법 알려줘"},
    {"AIMessage": "자동차보험료는 다음과 같이 계산됩니다..."}
  ],
  "citations": [
    {"title": "보험료계산서.pdf", "page": 15},
    {"title": "자동차보험가이드.pdf", "page": 23}
  ]
}
```

**Response (200 OK) - Multipart Response:**
```
HTTP/1.1 200 OK
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxkTrZu0gW
X-RAG-Success: true
X-Voice-Name: ko-KR-SunHiNeural
X-Citations-Count: 2
X-Audio-Format: wav
X-Response-Type: multipart

------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="json"
Content-Type: application/json

{
  "success": true,
  "messages": [
    {"HumanMessage": "자동차보험료 계산 방법 알려줘"},
    {"AIMessage": "자동차보험료는 다음과 같이 계산됩니다..."}
  ],
  "citations": [
    {"title": "보험료계산서.pdf", "page": 15},
    {"title": "자동차보험가이드.pdf", "page": 23}
  ]
}
------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="audio"; filename="answer.wav"
Content-Type: audio/wav

[WAV 오디오 파일 바이너리 데이터]
------WebKitFormBoundary7MA4YWxkTrZu0gW--
```

**6. POST /tts/convert-rag-response-file - RAG 응답을 WAV 파일로 직접 다운로드**
**WAV 파일 직접 반환**

**Request DTO:** 동일한 RAG 응답 구조

**Response (200 OK) - WAV 파일:**
```
HTTP/1.1 200 OK
Content-Type: audio/wav
Content-Disposition: attachment; filename="rag_answer.wav"
X-Voice-Name: ko-KR-SunHiNeural
X-Text-Length: 125
X-RAG-Success: true
X-Citations-Count: 2
X-First-Citation: 보험료계산서.pdf::15
X-TTS-Success: true
X-AI-Message-Preview: 자동차보험료는 다음과 같이 계산됩니다...
X-RAG-Messages-Count: 2

[WAV 오디오 파일 바이너리 데이터]
```



## STT Service API 계약

### 서비스 개요
- **서비스명**: STT Service
- **설명**: 음성을 텍스트로 변환하는 서비스
- **기술**: Azure Cognitive Services, FastAPI

### API 엔드포인트

**1. POST /stt/convert - 음성을 텍스트로 변환**
**오디오 파일을 텍스트로 변환하는 메인 엔드포인트**

**Request DTO:**
```
POST /stt/convert
Content-Type: multipart/form-data

Form Fields:
- audio_file: WAV 파일 (필수)
- locale: 언어 코드 (선택사항, 기본값: ko-KR)
```

**Response (200 OK):**
```json
{
  "success": true,
  "text": "변환된 전체 텍스트 내용입니다.",
  "segments": [
    "첫 번째 세그먼트",
    "두 번째 세그먼트"
  ],
  "locale": "ko-KR",
  "confidence": 0.95,
  "processing_time": 2.34
}
```

**응답 필드 설명:**
| 필드 | 타입 | 설명 |
|------|------|------|
| success | boolean | 변환 성공 여부 |
| text | string | 변환된 전체 텍스트 |
| segments | array | 텍스트 세그먼트 배열 |
| locale | string | 사용된 언어 코드 |
| confidence | float | 인식 신뢰도 (0-1) |
| processing_time | float | 처리 시간 (초) |

**2. GET /health - 서비스 상태 확인**
**서비스 상태 확인 엔드포인트**

**Response (200 OK):**
```json
{
  "status": "healthy",
  "service": "STT Service",
  "version": "1.0.0",
  "uptime": "2 days, 14:32:15",
  "azure_connection": "connected"
}
```

**3. GET /docs - 대화형 API 문서**
**Swagger UI를 통한 대화형 API 문서**

**4. GET /redoc - 대안 API 문서**
**ReDoc을 통한 대안 API 문서**



## Sound Q&A API 계약

### 서비스 개요
- **서비스명**: Sound Q&A
- **설명**: 음성 질문을 입력으로 받아 텍스트와 음성으로 답변을 돌려주는 서비스
- **기술**: Azure API Management, FastAPI

### API 엔드포인트

**1. POST /soundqna/qna - 음성을 텍스트로 변환**
**오디오 파일을 텍스트로 변환하는 메인 엔드포인트**

**Request DTO:**
```
POST /soundqna/qna
Content-Type: multipart/form-data

Form Fields:
- audio_file: WAV 파일 (필수)
- locale: 언어 코드 (선택사항, 기본값: ko-KR)
```

**Response (200 OK):**
```
HTTP/1.1 200 OK
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxkTrZu0gW
X-RAG-Success: true
X-Voice-Name: ko-KR-SunHiNeural
X-Citations-Count: 2
X-Audio-Format: wav
X-Response-Type: multipart

------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="json"
Content-Type: application/json

{
  "success": true,
  "messages": [
    {"HumanMessage": "자동차보험료 계산 방법 알려줘"},
    {"AIMessage": "자동차보험료는 다음과 같이 계산됩니다..."}
  ],
  "citations": [
    {"title": "보험료계산서.pdf", "page": 15},
    {"title": "자동차보험가이드.pdf", "page": 23}
  ]
}
------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="audio"; filename="answer.wav"
Content-Type: audio/wav

[WAV 오디오 파일 바이너리 데이터]
------WebKitFormBoundary7MA4YWxkTrZu0gW--
```
---

*이 문서는 KT AICC 기반 RAG 상담 지원 서비스의 모든 API 계약 명세를 다룹니다.*
