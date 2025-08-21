# 📋 ADR (Architecture Decision Records)

이 디렉토리는 KT AICC 기반 RAG 상담 지원 서비스의 주요 아키텍처 결정사항을 기록한다.

## 📚 ADR 목록

| 번호 | 제목 | 상태 | 설명 |
|------|------|------|------|
| [ADR-001](./adr-001-rag-architecture-design.md) | RAG 시스템 아키텍처 설계 방향 | 초기 결정 | RAG 모델을 여러 단계로 분할하는 방향으로 결정 |
| [ADR-002](./adr-002-rag-integrated-architecture.md) | RAG 시스템 통합 아키텍처 채택 | 방향 전환 | 시간적 제약과 응답 효율성을 고려하여 RAG 모델을 하나로 통합 |
| [ADR-003](./adr-003-orchestrator-pattern.md) | 마이크로서비스 접근 방식 - 오케스트레이터 패턴 채택 | 승인됨 | 오케스트레이터 패턴을 활용하여 파편화된 서비스들에 대한 단일 진입점 제공 |
| [ADR-004](./adr-004-mongodb-schema-separation.md) | MongoDB 스키마 분리 - 단일 컬렉션에서 이원화 구조로 전환 | 승인됨 | products와 documents를 별도 컬렉션으로 분리하여 RAG 시스템 최적화 |

## 🔄 ADR 흐름

```
ADR-001: RAG 분할 설계
    ↓ (실제 구현 과정에서 재검토)
ADR-002: RAG 통합 아키텍처
    ↓ (서비스 접근 방식 결정)
ADR-003: 오케스트레이터 패턴
    ↓ (데이터베이스 구조 최적화)
ADR-004: MongoDB 스키마 분리
```

## 📝 ADR 작성 가이드

각 ADR은 다음 구조를 따른다:

1. **Context**: 결정이 필요한 상황과 배경
2. **Alternatives**: 고려한 대안들과 각각의 장단점
3. **Decision**: 최종 결정과 그 이유
4. **Consequences**: 결정의 결과와 영향

## 🔗 관련 문서

- [메인 README](../README.md): 시스템 전체 개요
- [API 계약 명세](../api-contracts.md): 서비스별 API 상세 정보
- [MongoDB 스키마](../mongodb_schema.md): 데이터베이스 구조 상세 정보
