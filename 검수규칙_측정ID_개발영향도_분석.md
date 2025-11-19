# 검수 규칙 측정ID(RuleID) 체계 도입 개발 영향도 분석

## 1. 영향도 요약

### 전체 영향도: **중간 (Medium)**

- **코드 변경 범위**: 중간
- **데이터 마이그레이션**: 필요
- **호환성 유지**: 가능 (OriginalRuleId로 보존)
- **테스트 범위**: 중간
- **배포 리스크**: 낮음

## 2. 영향받는 컴포넌트 분석

### 2.1 모델 클래스 (Models/Config)

#### 영향도: **낮음 (Low)**

**영향받는 파일:**
- `AttributeCheckConfig.cs` ✅ (이미 RuleId 속성 있음)
- `RelationCheckConfig.cs` ✅ (이미 RuleId 속성 있음)
- `TableCheckConfig.cs` ⚠️ (RuleId 속성 추가 필요)
- `SchemaCheckConfig.cs` ⚠️ (RuleId 속성 추가 필요)
- `GeometryCheckConfig.cs` ⚠️ (RuleId 속성 추가 필요)

**필요한 변경:**
```csharp
// TableCheckConfig.cs에 추가
[Name("RuleId")]
public string RuleId { get; set; } = string.Empty;

[Name("OriginalRuleId")]
public string? OriginalRuleId { get; set; } // 호환성 유지용

[Name("Severity")]
public string? Severity { get; set; } = "MAJOR";
```

**작업량**: 각 파일당 약 5줄 추가, **총 15줄**

### 2.2 CSV 로더 (Services)

#### 영향도: **낮음 (Low)**

**영향받는 파일:**
- `CsvConfigService.cs` - CSV 파일 로드 로직
- `SimpleValidationService.cs` - CSV 로드 메서드들

**현재 상태:**
- CsvHelper를 사용하여 동적으로 CSV 매핑
- `[Name("RuleId")]` 속성으로 자동 매핑됨
- RuleId가 없어도 동작 (빈 문자열 처리)

**필요한 변경:**
1. RuleId 유효성 검증 로직 추가 (선택사항)
2. OriginalRuleId 컬럼 지원 (선택사항)

**작업량**: 약 20줄 추가 (유효성 검증 로직)

### 2.3 검수 프로세서 (Processors)

#### 영향도: **낮음 (Low)**

**영향받는 파일:**
- `AttributeCheckProcessor.cs` ✅ (이미 RuleId 사용)
- `RelationCheckProcessor.cs` ✅ (이미 RuleId 사용)
- `TableCheckProcessor.cs` ⚠️ (RuleId 로깅 추가)
- `SchemaCheckProcessor.cs` ⚠️ (RuleId 로깅 추가)
- `GeometryCheckProcessor.cs` ⚠️ (RuleId 로깅 추가)

**현재 사용 패턴:**
```csharp
// RelationCheckProcessor.cs 예시
_logger.LogInformation("관계 검수 시작: RuleId={RuleId}", config.RuleId);
RaiseProgress(config.RuleId ?? string.Empty, ...);
```

**필요한 변경:**
1. 1, 2, 3단계 프로세서에 RuleId 로깅 추가
2. 진행률 보고에 RuleId 포함 (이미 대부분 구현됨)

**작업량**: 각 프로세서당 약 5-10줄, **총 30줄**

### 2.4 오류 모델 (Models)

#### 영향도: **낮음 (Low)**

**영향받는 파일:**
- `ValidationError.cs`
- `AttributeRelationError.cs`
- `SpatialRelationError.cs`
- `QcError.cs` (데이터베이스 엔티티)

**현재 상태:**
- `ErrorCode` 속성 사용 (CheckType 기반)
- RuleId를 직접 저장하는 필드 없음

**필요한 변경:**
```csharp
// ValidationError.cs에 추가
public string RuleId { get; set; } = string.Empty;
public string? OriginalRuleId { get; set; } // 호환성 유지
```

**작업량**: 각 모델당 약 2줄, **총 8줄**

### 2.5 데이터베이스 스키마

#### 영향도: **중간 (Medium)**

**영향받는 테이블:**
- `QcError` 테이블 (ValidationDbContext)

**현재 스키마:**
```csharp
public string ErrorCode { get; set; } // CheckType 기반
```

**필요한 변경:**
```csharp
// QcError.cs에 추가
public string RuleId { get; set; } = string.Empty; // 새 RuleId
public string? OriginalRuleId { get; set; } // 기존 RuleId (호환성)
```

**마이그레이션 필요:**
- 기존 데이터의 ErrorCode → OriginalRuleId 변환
- 새 RuleId 컬럼 추가 (NULL 허용, 이후 점진적 채움)

**작업량**: 
- 엔티티 수정: 약 5줄
- 마이그레이션 스크립트: 약 50줄
- 데이터 변환 스크립트: 약 100줄

### 2.6 CSV 파일 구조

#### 영향도: **중간 (Medium)**

**영향받는 파일:**
- `1_table_check.csv` - RuleId 컬럼 추가 필요
- `2_schema_check.csv` - RuleId 컬럼 추가 필요
- `3_geometry_check.csv` - RuleId 컬럼 추가 필요
- `4_attribute_check.csv` - RuleId 변환 필요 (118개 규칙)
- `5_relation_check.csv` - RuleId 변환 필요 (50개 규칙)
- `codelist.csv` - RuleId 컬럼 추가 (선택사항)
- `geometry_criteria.csv` - RuleId 컬럼 추가 (선택사항)

**필요한 작업:**
1. CSV 파일 헤더 수정
2. 기존 데이터에 RuleId 추가 (변환 매핑표 사용)
3. OriginalRuleId 컬럼 추가 (호환성)

**작업량**: 
- CSV 파일 수정: 수동 작업 또는 스크립트 사용
- 변환 스크립트 작성: 약 200줄

### 2.7 UI 컴포넌트 (GUI)

#### 영향도: **낮음 (Low)**

**영향받는 파일:**
- `ValidationResultView.xaml.cs` - 오류 표시
- `MainWindow.xaml.cs` - 검수 설정
- 오류 목록 표시 관련 ViewModel

**현재 상태:**
- RuleId를 직접 표시하지 않음
- ErrorCode 기반으로 오류 분류

**필요한 변경:**
1. 오류 목록에 RuleId 컬럼 추가 (선택사항)
2. RuleId 기반 필터링 기능 추가 (선택사항)

**작업량**: 약 30줄 (UI 개선, 선택사항)

### 2.8 로깅 및 추적

#### 영향도: **낮음 (Low)**

**현재 상태:**
- RuleId는 이미 로깅에 사용됨
- 진행률 보고에 RuleId 포함됨

**필요한 변경:**
- 1, 2, 3단계에도 RuleId 로깅 추가

**작업량**: 약 15줄

## 3. 단계별 개발 작업량 추정

### Phase 1: 모델 및 CSV 구조 변경 (1주)

| 작업 항목 | 파일 수 | 예상 코드 라인 | 난이도 |
|---------|--------|--------------|--------|
| Config 모델에 RuleId 추가 | 3개 | 15줄 | 낮음 |
| ValidationError 모델 수정 | 4개 | 8줄 | 낮음 |
| CSV 파일 구조 변경 | 7개 | 수동/스크립트 | 중간 |
| CSV 변환 스크립트 작성 | 1개 | 200줄 | 중간 |
| **소계** | **15개** | **223줄** | **중간** |

### Phase 2: 프로세서 수정 (1주)

| 작업 항목 | 파일 수 | 예상 코드 라인 | 난이도 |
|---------|--------|--------------|--------|
| TableCheckProcessor 수정 | 1개 | 10줄 | 낮음 |
| SchemaCheckProcessor 수정 | 1개 | 10줄 | 낮음 |
| GeometryCheckProcessor 수정 | 1개 | 10줄 | 낮음 |
| 오류 생성 시 RuleId 포함 | 3개 | 15줄 | 낮음 |
| **소계** | **6개** | **45줄** | **낮음** |

### Phase 3: 데이터베이스 마이그레이션 (3일)

| 작업 항목 | 파일 수 | 예상 코드 라인 | 난이도 |
|---------|--------|--------------|--------|
| QcError 엔티티 수정 | 1개 | 5줄 | 낮음 |
| 마이그레이션 스크립트 | 1개 | 50줄 | 중간 |
| 데이터 변환 스크립트 | 1개 | 100줄 | 중간 |
| **소계** | **3개** | **155줄** | **중간** |

### Phase 4: 테스트 및 검증 (1주)

| 작업 항목 | 파일 수 | 예상 코드 라인 | 난이도 |
|---------|--------|--------------|--------|
| 단위 테스트 작성/수정 | 10개 | 200줄 | 중간 |
| 통합 테스트 | - | - | 중간 |
| CSV 파일 검증 | - | - | 낮음 |
| **소계** | **10개** | **200줄** | **중간** |

### 총 작업량 요약

| Phase | 파일 수 | 코드 라인 | 예상 기간 |
|------|---------|----------|----------|
| Phase 1 | 15개 | 223줄 | 1주 |
| Phase 2 | 6개 | 45줄 | 1주 |
| Phase 3 | 3개 | 155줄 | 3일 |
| Phase 4 | 10개 | 200줄 | 1주 |
| **합계** | **34개** | **623줄** | **약 3주** |

## 4. 리스크 분석

### 4.1 기술적 리스크

#### 높은 리스크 (High Risk)
- **없음**

#### 중간 리스크 (Medium Risk)
1. **CSV 파일 변환 오류**
   - 기존 RuleId → 새 RuleId 매핑 실수 가능성
   - **완화 방안**: 자동화 스크립트 + 검증 로직
   - **확률**: 낮음

2. **데이터베이스 마이그레이션 실패**
   - 기존 데이터 손실 가능성
   - **완화 방안**: 백업 + 트랜잭션 처리
   - **확률**: 매우 낮음

#### 낮은 리스크 (Low Risk)
1. **호환성 문제**
   - 기존 코드가 RuleId를 하드코딩한 경우
   - **확률**: 매우 낮음 (동적 로드 사용)

2. **성능 영향**
   - RuleId 추가로 인한 성능 저하
   - **확률**: 없음 (문자열 필드 추가만)

### 4.2 비즈니스 리스크

#### 높은 리스크
- **없음**

#### 중간 리스크
1. **사용자 혼란**
   - RuleId 변경으로 인한 사용자 혼란
   - **완화 방안**: OriginalRuleId로 호환성 유지 + 문서화
   - **확률**: 낮음

#### 낮은 리스크
1. **검수 결과 불일치**
   - 기존 검수 결과와 새 RuleId 기반 결과 비교 필요
   - **확률**: 없음 (동일한 규칙, ID만 변경)

## 5. 호환성 유지 전략

### 5.1 하위 호환성

**전략:**
1. **OriginalRuleId 컬럼 추가**
   - 기존 RuleId를 보존하여 호환성 유지
   - 기존 코드는 OriginalRuleId 참조 가능

2. **점진적 전환**
   - Phase 1: OriginalRuleId와 새 RuleId 병행 사용
   - Phase 2: 새 RuleId로 전환, OriginalRuleId는 참조용으로 유지
   - Phase 3: OriginalRuleId 제거 (선택사항)

3. **기본값 처리**
   - RuleId가 없는 경우 자동 생성 또는 기본값 사용
   - 기존 CSV 파일도 정상 동작

### 5.2 마이그레이션 경로

```
기존 시스템 (RuleId 없음/혼재)
    ↓
Phase 1: OriginalRuleId 보존 + 새 RuleId 추가
    ↓
Phase 2: 새 RuleId로 전환 (OriginalRuleId 참조용)
    ↓
Phase 3: OriginalRuleId 제거 (선택사항)
```

## 6. 테스트 전략

### 6.1 단위 테스트

**필요한 테스트:**
1. RuleId 유효성 검증 테스트
2. CSV 로더 RuleId 매핑 테스트
3. 기존 RuleId → 새 RuleId 변환 테스트
4. OriginalRuleId 보존 테스트

**예상 테스트 케이스 수**: 약 20개

### 6.2 통합 테스트

**필요한 테스트:**
1. 전체 검수 프로세스 RuleId 포함 테스트
2. 데이터베이스 저장/조회 테스트
3. 오류 리포트 RuleId 포함 테스트

**예상 테스트 케이스 수**: 약 10개

### 6.3 회귀 테스트

**필요한 테스트:**
1. 기존 검수 규칙 동작 확인
2. 기존 CSV 파일 호환성 확인
3. 기존 데이터베이스 호환성 확인

**예상 테스트 케이스 수**: 약 15개

## 7. 배포 계획

### 7.1 배포 단계

#### Stage 1: 개발 환경 배포
- 목적: 개발 및 테스트
- 기간: 1주
- 리스크: 낮음

#### Stage 2: 스테이징 환경 배포
- 목적: 통합 테스트 및 검증
- 기간: 1주
- 리스크: 낮음

#### Stage 3: 프로덕션 배포
- 목적: 실제 운영 환경 적용
- 기간: 1일
- 리스크: 중간 (롤백 계획 필요)

### 7.2 롤백 계획

**롤백 조건:**
- RuleId 변환 오류 발생
- 데이터베이스 마이그레이션 실패
- 검수 결과 불일치

**롤백 절차:**
1. OriginalRuleId 기반으로 복원
2. 데이터베이스 롤백 (트랜잭션 사용)
3. CSV 파일 이전 버전으로 복원

## 8. 예상 일정

### 전체 일정: **약 3-4주**

| 주차 | 작업 내용 | 산출물 |
|-----|---------|--------|
| 1주차 | Phase 1: 모델 및 CSV 구조 변경 | 수정된 모델, CSV 파일 |
| 2주차 | Phase 2: 프로세서 수정 | 수정된 프로세서 코드 |
| 3주차 | Phase 3: DB 마이그레이션 + Phase 4: 테스트 | 마이그레이션 스크립트, 테스트 결과 |
| 4주차 | 버그 수정 및 최종 검증 | 배포 준비 완료 |

## 9. 권장 사항

### 9.1 즉시 적용 가능 (Quick Win)

1. **1, 2, 3단계 RuleId 추가**
   - 영향도 낮음, 즉시 적용 가능
   - 기존 코드 변경 최소화

2. **OriginalRuleId 컬럼 추가**
   - 호환성 유지, 리스크 최소화

### 9.2 점진적 적용

1. **Phase별 적용**
   - 각 Phase 완료 후 검증
   - 문제 발생 시 즉시 중단 가능

2. **기능 플래그 사용**
   - 새 RuleId 사용 여부를 설정으로 제어
   - 문제 발생 시 즉시 비활성화 가능

### 9.3 장기적 개선

1. **RuleId 관리 도구 개발**
   - 규칙 관리 대시보드
   - RuleId 자동 생성 도구

2. **문서화 자동화**
   - RuleId 기반 문서 자동 생성
   - 규칙 변경 이력 자동 추적

## 10. 결론

### 전체 평가

- **개발 난이도**: 중간
- **작업량**: 중간 (약 600줄 코드)
- **리스크**: 낮음-중간
- **비즈니스 가치**: 높음 (관리 효율성 향상)

### 권장 사항

✅ **즉시 진행 가능**: 영향도가 낮고 리스크가 낮은 작업
✅ **점진적 적용**: Phase별로 나누어 안전하게 적용
✅ **호환성 유지**: OriginalRuleId로 기존 시스템과 호환성 보장

### 다음 단계

1. **승인 및 계획 수립** (1일)
2. **Phase 1 시작** (1주)
3. **단계별 검토 및 승인** (각 Phase 완료 시)

---

**작성일**: 2025-11-18  
**작성자**: SpatialCheckPro 개발팀  
**버전**: 1.0

