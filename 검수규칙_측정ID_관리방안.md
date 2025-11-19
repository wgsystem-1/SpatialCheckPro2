# 검수 규칙 측정ID(RuleID) 체계적 관리 방안

## 1. 현재 상태 분석

### 1.1 현재 구조
- **1단계 (테이블 검수)**: 측정ID 없음
- **2단계 (스키마 검수)**: 측정ID 없음  
- **3단계 (지오메트리 검수)**: 측정ID 없음
- **4단계 (속성 관계 검수)**: RuleId 있음 (118개 규칙)
- **5단계 (공간 관계 검수)**: RuleId 있음 (50개 규칙)
- **코드리스트 (codelist.csv)**: 측정ID 없음
- **지오메트리 기준 (geometry_criteria.csv)**: 측정ID 없음

### 1.2 문제점
1. **일관성 부족**: 4, 5단계만 RuleId가 있고 나머지는 없음
2. **명명 규칙 혼재**: 영문(ATTR_CODE_BULD)과 한글(건물구분_Code) 혼용
3. **분류 체계 부재**: 단계별/유형별 체계적 분류 없음
4. **확장성 부족**: 새로운 규칙 추가 시 ID 충돌 가능성
5. **추적성 부족**: 규칙 변경 이력 관리 어려움

## 2. ISO 19157 기반 측정ID 체계 설계

### 2.0 ISO 19157 품질요소 5가지와 검수 단계 매핑

ISO 19157:2013 Geographic information — Data quality 표준의 품질요소 5가지를 검수 단계와 매핑:

| 품질요소 (Quality Element) | 검수 단계 | 주요 RuleID 카테고리 | 설명 |
|---------------------------|----------|-------------------|------|
| **1. 완전성 (Completeness)** | 1단계 (테이블 검수) | TB, FC | 누락/과잉 데이터 검사, 필수 피처클래스 존재 여부 |
| **2. 논리적 일관성 (Logical Consistency)** | 2단계 (스키마 검수)<br>3단계 (지오메트리 검수)<br>5단계 (공간 관계 검수) | FD, DT, UK, FK, NN<br>DP, OV, SI, SO, HO<br>PI, LW, PN, LC, CA | 개념/도메인/포맷/위상 일관성, 위상 관계 검증 |
| **3. 위치 정확도 (Positional Accuracy)** | 3단계 (지오메트리 검수) | SH, SA, SL, SP, US, OS | 절대/상대 위치 정확도, 기하 정확도 검사 |
| **4. 시간 정확도 (Temporal Accuracy)** | 4단계 (속성 관계 검수) | TM | 시간 측정 정확도 (향후 확장 가능) |
| **5. 주제 정확도 (Thematic Accuracy)** | 4단계 (속성 관계 검수) | CD, RG, RN, IF | 속성 분류 정확도, 코드값 검증 |

**RuleID 카테고리와 품질요소 매핑:**
- **완전성**: `1TB*`, `1FC*` (테이블/피처클래스 존재)
- **논리적 일관성**: `2FD*`, `2DT*`, `2UK*`, `2FK*`, `3DP*`, `3OV*`, `3SI*`, `5PI*`, `5LW*`, `5PN*`
- **위치 정확도**: `3SH*`, `3SA*`, `3SL*`, `3SP*`, `3US*`, `3OS*`
- **시간 정확도**: `4TM*` (향후 확장)
- **주제 정확도**: `4CD*`, `4RG*`, `4RN*`, `4IF*`

### 2.1 측정ID 구조 (ISO 19157-1:2023 기반)

**구조 4: 하이브리드 (권장)**
```
[STAGE][ISO_SUB_ELEMENT][TYPE][SEQUENCE]
```

**구성 요소:**
- **STAGE**: 검수 단계 (1자리 숫자)
  - `1`: 테이블 검수
  - `2`: 스키마 검수
  - `3`: 지오메트리 검수
  - `4`: 속성 관계 검수
  - `5`: 공간 관계 검수
- **ISO_SUB_ELEMENT**: ISO 19157-1:2023 세부 항목 Abbreviation (2자리 영문 대문자)
  - 완전성: `OM` (Omission), `CM` (Commission)
  - 논리 일관성: `CC` (Conceptual Consistency), `DC` (Domain Consistency), `FC` (Format Consistency), `TC` (Topological Consistency)
  - 위치 정확도: `AP` (Absolute Positional Accuracy), `RP` (Relative Positional Accuracy), `GP` (Gridded Positional Accuracy)
  - 시간 품질: `AM` (Accuracy of Time Measurement), `TC` (Temporal Consistency), `TV` (Temporal Validity)
  - 주제 품질: `CC` (Classification Correctness), `NA` (Non-quantitative Attribute), `QA` (Quantitative Attribute)
- **TYPE**: 검수 유형 (2자리 영문 대문자)
- **SEQUENCE**: 일련번호 (3자리 숫자, 001~999)

**예시:**
- `3-TC-DP-001`: 3단계-위상 일관성-중복-001
- `3-RP-SH-001`: 3단계-상대 위치 정확도-짧은객체-001
- `4-CC-CD-001`: 4단계-분류 정확성-코드리스트-001

**참고:** 품질요소 정보는 메타데이터로 관리 (별도 CSV/DB 테이블)

### 2.2 ISO 19157-1:2023 세부 항목 Abbreviation 매핑

#### 1단계: 테이블 검수 (STAGE=1) - 완전성 (Completeness)
**ISO 세부 항목: Omission (누락)**
- `OM-TB`: Table (테이블) - 테이블 존재 여부
- `OM-FC`: FeatureClass (피처클래스) - 피처클래스 존재 여부
- `OM-CR`: CRS (좌표계) - 좌표계 지정 여부
- `OM-GT`: GeometryType (지오메트리 타입) - 지오메트리 타입 일치 여부

#### 2단계: 스키마 검수 (STAGE=2) - 논리 일관성 (Logical Consistency)
**ISO 세부 항목: Format Consistency (형식 일관성), Domain Consistency (도메인 일관성)**
- `FC-FD`: FieldDefinition (필드 정의) - 필드 존재 여부
- `FC-DT`: DataType (데이터 타입) - 데이터 타입 일치
- `FC-LN`: Length (길이) - 필드 길이 일치
- `FC-PR`: Precision (정밀도) - 정밀도 일치
- `DC-UK`: UniqueKey (고유키) - 고유키 중복 검사
- `DC-FK`: ForeignKey (외래키) - 외래키 참조 무결성
- `DC-NN`: NotNull (필수값) - 필수값 NULL 검사

#### 3단계: 지오메트리 검수 (STAGE=3)

**논리 일관성 - 형식 일관성 (Format Consistency)**
- `FC-VL`: Validity (유효성) - IsValid 검사
- `FC-EM`: Empty (빈 지오메트리) - IsEmpty 검사
- `FC-NU`: Null (NULL 지오메트리) - NULL 검사
- `FC-SM`: Simple (단순성) - IsSimple 검사

**논리 일관성 - 위상 일관성 (Topological Consistency)**
- `TC-DP`: Duplicate (중복) - 객체 중복 검사
- `TC-OV`: Overlap (겹침) - 객체 간 겹침 검사
- `TC-SI`: SelfIntersection (자기교차) - 자체 꼬임 검사
- `TC-SO`: SelfOverlap (자기중첩) - 자기 중첩 검사
- `TC-HO`: Hole (홀) - 홀 폴리곤 오류 검사
- `TC-VN`: Vertex (정점) - 최소 정점 개수 검사
- `TC-RG`: Ring (링) - 링 폐합 검사
- `TC-OR`: Orientation (방향) - 링 방향성 검사

**위치 정확도 - 상대 위치 정확도 (Relative Positional Accuracy)**
- `RP-SH`: Short (짧은 객체) - 최소 선 길이 검사
- `RP-SA`: Small (작은 면적) - 최소 면적 검사
- `RP-SL`: Sliver (슬리버) - 슬리버 폴리곤 검사
- `RP-SP`: Spike (스파이크) - 스파이크 검사
- `RP-US`: Undershoot (언더슛) - 언더슛 검사
- `RP-OS`: Overshoot (오버슛) - 오버슛 검사

#### 4단계: 속성 관계 검수 (STAGE=4)

**주제 품질 - 분류 정확성 (Classification Correctness)**
- `CC-CD`: CodeList (코드리스트) - 코드값 유효성 검사
- `CC-IF`: IfThen (조건부) - 조건부 속성 검사

**주제 품질 - 비정량 속성 정확성 (Non-quantitative Attribute Correctness)**
- `NA-RG`: Regex (정규식) - 형식 일치 검사
- `NA-TY`: Typo (오타) - 오타 검사

**주제 품질 - 정량 속성 정확도 (Quantitative Attribute Accuracy)**
- `QA-RN`: Range (범위) - 값 범위 검사
- `QA-NZ`: NotZero (0 금지) - 0 값 금지 검사
- `QA-EQ`: Equals (일치) - 값 일치 검사
- `QA-NE`: NotEquals (불일치) - 값 불일치 검사
- `QA-GT`: GreaterThan (초과) - 값 초과 검사
- `QA-GE`: GreaterThanOrEqual (이상) - 값 이상 검사
- `QA-LT`: LessThan (미만) - 값 미만 검사
- `QA-LE`: LessThanOrEqual (이하) - 값 이하 검사
- `QA-MO`: MultipleOf (배수) - 배수 검사

**논리 일관성 - 도메인 일관성 (Domain Consistency)**
- `DC-NO`: NotNull (필수값) - 필수 속성 NULL 검사

**시간 품질 - 시간 측정 정확도 (Accuracy of Time Measurement)**
- `AM-TM`: Temporal (시간) - 시간 측정 정확도 (향후 확장)

#### 5단계: 공간 관계 검수 (STAGE=5)

**논리 일관성 - 위상 일관성 (Topological Consistency)**
- `TC-PI`: PointInsidePolygon (점-면 포함) - 점이 폴리곤 내부에 있는지 검사
- `TC-LW`: LineWithinPolygon (선-면 포함) - 선이 폴리곤 내부에 있는지 검사
- `TC-PW`: PolygonWithinPolygon (면-면 포함) - 폴리곤이 다른 폴리곤 내부에 있는지 검사
- `TC-PN`: PolygonNotOverlap (면-면 겹침 금지) - 폴리곤 간 겹침 금지 검사
- `TC-LI`: LineIntersection (선 교차) - 선 교차 검사
- `TC-PX`: PolygonIntersection (면 교차) - 폴리곤 교차 검사
- `TC-LC`: LineConnectivity (선 연결성) - 선 연결성 검사
- `TC-LD`: LineDisconnection (선 단절) - 선 단절 검사
- `TC-DC`: DefectiveConnection (결함 연결) - 결함 연결 검사
- `TC-EP`: Endpoint (끝점) - 끝점 위치 검사
- `TC-CA`: CenterlineAttribute (중심선 속성) - 중심선 속성 불일치 검사
- `TC-BM`: BoundaryMatch (경계 일치) - 경계 일치 검사

**위치 정확도 - 상대 위치 정확도 (Relative Positional Accuracy)**
- `RP-PS`: PointSpacing (점 간격) - 점 간 최소 거리 검사

### 2.3 측정ID 예시

#### 1단계 예시
- `1TB001`: 테이블 존재 여부 검사
- `1FC001`: 필수 피처클래스 존재 여부
- `1CR001`: 좌표계 지정 여부
- `1GT001`: 지오메트리 타입 일치 여부

#### 2단계 예시
- `2FD001`: 필드 존재 여부
- `2DT001`: 데이터 타입 일치
- `2UK001`: 고유키 중복 검사
- `2FK001`: 외래키 참조 무결성
- `2NN001`: 필수값 NULL 검사

#### 3단계 예시
**논리 일관성 - 형식 일관성**
- `3-FC-VL-001`: 지오메트리 유효성 검사 (IsValid)
- `3-FC-EM-001`: 빈 지오메트리 검사 (IsEmpty)
- `3-FC-NU-001`: NULL 지오메트리 검사
- `3-FC-SM-001`: 지오메트리 단순성 검사 (IsSimple)

**논리 일관성 - 위상 일관성**
- `3-TC-DP-001`: 객체 중복 검사
- `3-TC-OV-001`: 객체 간 겹침 검사
- `3-TC-SI-001`: 자체 꼬임 검사 (자기교차)
- `3-TC-SO-001`: 자기 중첩 검사
- `3-TC-HO-001`: 홀 폴리곤 오류 검사
- `3-TC-VN-001`: 최소 정점 개수 검사
- `3-TC-RG-001`: 링 폐합 검사
- `3-TC-OR-001`: 링 방향성 검사

**위치 정확도 - 상대 위치 정확도**
- `3-RP-SH-001`: 짧은 객체 검사 (최소 선 길이)
- `3-RP-SA-001`: 작은 면적 객체 검사 (최소 면적)
- `3-RP-SL-001`: 슬리버 검사
- `3-RP-SP-001`: 스파이크 검사
- `3-RP-US-001`: 언더슛 검사
- `3-RP-OS-001`: 오버슛 검사

#### 4단계 예시
**주제 품질 - 분류 정확성**
- `4-CC-CD-001`: 건물구분 코드리스트 검사
- `4-CC-IF-001`: 조건부 검사 (도로구분별 도로폭)

**주제 품질 - 비정량 속성 정확성**
- `4-NA-RG-001`: PNU 정규식 검사

**주제 품질 - 정량 속성 정확도**
- `4-QA-RN-001`: 표고점높이 범위 검사
- `4-QA-NZ-001`: 층수 0 금지
- `4-QA-MO-001`: 등고선높이 5의 배수

**논리 일관성 - 도메인 일관성**
- `4-DC-NO-001`: 필수 속성 NULL 검사

#### 5단계 예시
**논리 일관성 - 위상 일관성**
- `5-TC-PI-001`: 건물중심점-건물 포함 관계
- `5-TC-LW-001`: 도로중심선-도로경계면 포함 관계
- `5-TC-PN-001`: 건물-도로경계면 겹침 금지
- `5-TC-LC-001`: 도로중심선 연결성 검사
- `5-TC-CA-001`: 도로중심선 속성 불일치 검사

**위치 정확도 - 상대 위치 정확도**
- `5-RP-PS-001`: 점 간 최소 거리 검사

## 3. CSV 파일 구조 개선안

### 3.1 1_table_check.csv 개선안

```csv
RuleId,Enabled,TableId,TableName,GeometryType,CRS,CheckType,Severity,Note
1TB001,Y,*,모든테이블,*,*,TableExists,MAJOR,필수 테이블 존재 여부 검사
1FC001,Y,tn_buld,건물,POLYGON,EPSG:5179,FeatureClassExists,MAJOR,건물 피처클래스 존재 여부
1CR001,Y,tn_buld,건물,POLYGON,EPSG:5179,CRSSpecified,MAJOR,좌표계 지정 여부
1GT001,Y,tn_buld,건물,POLYGON,EPSG:5179,GeometryTypeMatch,MAJOR,지오메트리 타입 일치 여부
```

### 3.2 2_schema_check.csv 개선안

```csv
RuleId,Enabled,TableId,FieldName,FieldAlias,DataType,Length,UK,FK,NN,RefTable,RefColumn,CheckType,Severity,Note
2FD001,Y,tn_alpt,objectid,시스템고유아이디,Integer,,,,Y,,,FieldExists,MAJOR,필드 존재 여부
2DT001,Y,tn_alpt,ncid,변동정보관리아이디,String,20,Y,,,,,DataTypeMatch,MAJOR,데이터 타입 일치
2UK001,Y,tn_alpt,ncid,변동정보관리아이디,String,20,Y,,,,,UniqueKey,MAJOR,고유키 중복 검사
2NN001,Y,tn_alpt,objectid,시스템고유아이디,Integer,,,,Y,,,NotNull,MAJOR,필수값 NULL 검사
```

### 3.3 3_geometry_check.csv 개선안

**기본 검수 항목 추가 (ISO 19107 필수 검사)**

```csv
RuleId,Enabled,TableId,TableName,GeometryType,지오메트리유효성,NULL지오메트리,빈지오메트리,지오메트리단순성,객체중복,객체간겹침,자체꼬임,슬리버,짧은객체,작은면적객체,홀폴리곤오류,최소정점개수,스파이크,자기중첩,언더슛,오버슛,Severity,Note
3VL001,Y,tn_buld,건물,MULTIPOLYGON,Y,N,N,N,Y,Y,Y,Y,N,Y,Y,Y,Y,Y,Y,N,N,MAJOR,지오메트리 유효성 검사 (IsValid)
3NU001,Y,tn_buld,건물,MULTIPOLYGON,N,Y,N,N,Y,Y,Y,Y,N,Y,Y,Y,Y,Y,Y,N,N,MAJOR,NULL 지오메트리 검사
3EM001,Y,tn_buld,건물,MULTIPOLYGON,N,N,Y,N,Y,Y,Y,Y,N,Y,Y,Y,Y,Y,Y,N,N,MAJOR,빈 지오메트리 검사 (IsEmpty)
3SM001,Y,tn_buld,건물,MULTIPOLYGON,N,N,N,Y,Y,Y,Y,Y,N,Y,Y,Y,Y,Y,Y,N,N,MAJOR,지오메트리 단순성 검사 (IsSimple)
3DP001,Y,tn_buld,건물,MULTIPOLYGON,N,N,N,N,Y,N,N,N,N,N,N,N,N,N,N,N,N,MAJOR,객체 중복 검사
3OV001,Y,tn_buld,건물,MULTIPOLYGON,N,N,N,N,N,Y,N,N,N,N,N,N,N,N,N,N,N,MAJOR,객체 간 겹침 검사
3SI001,Y,tn_buld,건물,MULTIPOLYGON,N,N,N,N,N,N,Y,N,N,N,N,N,N,N,N,N,N,MAJOR,자체 꼬임 검사 (자기교차)
3SL001,Y,tn_buld,건물,MULTIPOLYGON,N,N,N,N,N,N,N,Y,N,N,N,N,N,N,N,N,N,MAJOR,슬리버 검사
3SH001,Y,tn_rodway_ctln,도로중심선,MULTILINESTRING,N,N,N,N,N,N,N,N,Y,N,N,N,N,N,N,N,N,MAJOR,짧은 객체 검사 (최소 선 길이)
3SA001,Y,tn_buld,건물,MULTIPOLYGON,N,N,N,N,N,N,N,N,N,Y,N,N,N,N,N,N,N,MAJOR,작은 면적 객체 검사 (최소 면적)
3HO001,Y,tn_buld,건물,MULTIPOLYGON,N,N,N,N,N,N,N,N,N,N,Y,N,N,N,N,N,N,MAJOR,홀 폴리곤 오류 검사
3VN001,Y,tn_buld,건물,MULTIPOLYGON,N,N,N,N,N,N,N,N,N,N,N,Y,N,N,N,N,N,MAJOR,최소 정점 개수 검사
3SP001,Y,tn_rodway_ctln,도로중심선,MULTILINESTRING,N,N,N,N,N,N,N,N,N,N,N,N,Y,N,N,N,N,MAJOR,스파이크 검사
3SO001,Y,tn_buld,건물,MULTIPOLYGON,N,N,N,N,N,N,N,N,N,N,N,N,N,Y,N,N,N,MAJOR,자기 중첩 검사
3US001,Y,tn_rodway_ctln,도로중심선,MULTILINESTRING,N,N,N,N,N,N,N,N,N,N,N,N,N,N,Y,N,N,MAJOR,언더슛 검사
3OS001,Y,tn_rodway_ctln,도로중심선,MULTILINESTRING,N,N,N,N,N,N,N,N,N,N,N,N,N,N,N,Y,N,MAJOR,오버슛 검사
```

**참고**: 기존 CSV 구조를 유지하면서 RuleId와 기본 검수 항목을 추가했습니다.

### 3.4 4_attribute_check.csv 개선안 (기존 RuleId 변환)

```csv
RuleId,Enabled,TableId,TableName,FieldName,CheckType,Parameters,Severity,Note,OriginalRuleId
4CD001,Y,tn_buld,건물,bldg_se,CodeList,건물구분V6,MAJOR,건물 용도 코드리스트,ATTR_CODE_BULD
4NZ001,Y,tn_buld,건물,bldg_nofl,NotZero,,MAJOR,층수 0 금지,ATTR_NOTZERO_FLOOR
4RG001,Y,tn_buld,건물,pnu,Regex,^[0-9]{19}$,MAJOR,PNU 19자리,ATTR_PNU_19LEN
4RN001,Y,tn_alpt,표고점,alpt_hgt,Range,0.0000001..1999.9999,MAJOR,표고점높이 0 금지 및 2000 미만,ATTR_ALPT_LT2000
```

### 3.5 5_relation_check.csv 개선안 (기존 RuleId 변환)

```csv
RuleId,Enabled,CaseType,MainTableId,MainTableName,RelatedTableId,RelatedTableName,FieldFilter,Tolerance,Severity,Note,OriginalRuleId
5PI001,Y,PointInsidePolygon,tn_buld,건물,tn_buld_ctpt,건물중심점,,,MAJOR,건물중심점은 건물을 벗어나면 안 됨,건물중심점_관계
5LW001,Y,LineWithinPolygon,tn_rodway_bndry,도로경계면,tn_rodway_ctln,도로중심선,road_se IN (...),0.001,MAJOR,중심선은 경계면을 벗어나면 안 됨,도로경계면_관계
5PN001,Y,PolygonNotOverlap,tn_buld,건물,tn_rodway_bndry,도로경계면,,0.1,MAJOR,겹침 면적 허용 0.1,건물_도로경계면_침범
```

### 3.6 codelist.csv 개선안

```csv
RuleId,CodeSetId,CodeValue,Label,Enabled,Severity,Note
4CD001,건물구분V6,BDG001,일반건물,Y,MAJOR,건물구분 코드리스트 검사
4CD001,건물구분V6,BDG002,공동주택,Y,MAJOR,건물구분 코드리스트 검사
```

### 3.7 geometry_criteria.csv 개선안

```csv
RuleId,항목명,값,단위,설명,Enabled,Severity
3OV001,겹침허용면적,0.01,제곱미터,폴리곤 겹침 허용 면적,Y,MAJOR
3SH001,최소선길이,0.01,미터,짧은 선 객체 판정 기준,Y,MAJOR
3SM001,최소폴리곤면적,0.1,제곱미터,작은 면적 객체 판정 기준,Y,MAJOR
3SI001,자체꼬임허용각도,1.0,도,자체 교차 허용 각도,Y,MAJOR
3SL001,슬리버면적,2.0,제곱미터,슬리버폴리곤 면적 기준,Y,MAJOR
3SP001,스파이크각도임계값,8.0,도,스파이크 검출 각도 임계값,Y,MAJOR
3US001,네트워크탐색거리,0.1,미터,언더슛/오버슛 탐색 거리,Y,MAJOR
```

## 4. 마이그레이션 전략

### 4.1 단계별 마이그레이션 계획

#### Phase 1: 기존 규칙 ID 변환 (1주)
1. 4단계 규칙 ID 변환 (118개)
   - 기존 RuleId → 새 RuleId 매핑 테이블 생성
   - `ATTR_CODE_BULD` → `4CD001`
   - `건물구분_Code` → `4CD002`
   
2. 5단계 규칙 ID 변환 (50개)
   - `건물중심점_관계` → `5PI001`
   - `도로경계면_관계` → `5LW001`

#### Phase 2: 신규 규칙 ID 추가 (2주)
1. 1단계 규칙 ID 추가
   - 테이블 존재 여부: `1TB001`
   - 필수 피처클래스: `1FC001`
   - 좌표계 검증: `1CR001`

2. 2단계 규칙 ID 추가
   - 필드 존재: `2FD001`
   - 데이터 타입: `2DT001`
   - 고유키: `2UK001`
   - 외래키: `2FK001`

3. 3단계 규칙 ID 추가
   - 각 검수 항목별로 고유 ID 부여
   - `3DP001` ~ `3OS001`

#### Phase 3: 코드리스트 및 기준 연계 (1주)
1. codelist.csv에 RuleId 추가
2. geometry_criteria.csv에 RuleId 추가
3. 규칙 간 참조 관계 설정

### 4.2 호환성 유지 방안

1. **기존 RuleId 유지**: `OriginalRuleId` 컬럼 추가로 기존 ID 보존
2. **점진적 전환**: 기존 코드는 OriginalRuleId 참조, 신규 코드는 새 RuleId 사용
3. **매핑 테이블**: 기존 ID → 새 ID 자동 변환 유틸리티 제공

## 5. 구현 방안

### 5.1 데이터베이스 스키마 확장

```sql
-- 규칙 마스터 테이블
CREATE TABLE validation_rules (
    rule_id VARCHAR(10) PRIMARY KEY,
    stage INTEGER NOT NULL,
    category VARCHAR(2) NOT NULL,
    type VARCHAR(2) NOT NULL,
    sequence INTEGER NOT NULL,
    original_rule_id VARCHAR(100),
    enabled BOOLEAN DEFAULT TRUE,
    severity VARCHAR(10),
    description TEXT,
    created_date DATETIME,
    updated_date DATETIME,
    version INTEGER DEFAULT 1
);

-- 규칙 변경 이력 테이블
CREATE TABLE validation_rule_history (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    rule_id VARCHAR(10) NOT NULL,
    change_type VARCHAR(20), -- CREATE, UPDATE, DELETE, ENABLE, DISABLE
    old_value TEXT,
    new_value TEXT,
    changed_by VARCHAR(100),
    changed_date DATETIME,
    FOREIGN KEY (rule_id) REFERENCES validation_rules(rule_id)
);
```

### 5.2 코드 변경 사항

1. **CheckIds.cs 확장**
```csharp
public static class CheckIds
{
    // ===== ISO 19157 품질요소 1: 완전성 (Completeness) =====
    // 1단계: 테이블 검수
    public const string TableExists = "1TB001";
    public const string FeatureClassExists = "1FC001";
    public const string CRSSpecified = "1CR001";
    public const string GeometryTypeMatch = "1GT001";
    
    // ===== ISO 19157 품질요소 2: 논리적 일관성 (Logical Consistency) =====
    // 2단계: 스키마 검수
    public const string FieldExists = "2FD001";
    public const string DataTypeMatch = "2DT001";
    public const string UniqueKeyCheck = "2UK001";
    public const string ForeignKeyCheck = "2FK001";
    public const string NotNullCheck = "2NN001";
    
    // 3단계: 지오메트리 검수 - 기본 검수
    public const string GeometryValidity = "3VL001";      // IsValid
    public const string GeometryNull = "3NU001";            // NULL
    public const string GeometryEmpty = "3EM001";          // IsEmpty
    public const string GeometrySimple = "3SM001";         // IsSimple
    
    // 3단계: 지오메트리 검수 - 위상 일관성
    public const string GeometryDuplicate = "3DP001";
    public const string GeometryOverlap = "3OV001";
    public const string SelfIntersection = "3SI001";
    public const string SelfOverlap = "3SO001";
    public const string HolePolygon = "3HO001";
    
    // 5단계: 공간 관계 검수 - 위상 관계
    public const string PointInsidePolygon = "5PI001";
    public const string LineWithinPolygon = "5LW001";
    public const string PolygonWithinPolygon = "5PW001";
    public const string PolygonNotOverlap = "5PN001";
    public const string LineConnectivity = "5LC001";
    public const string CenterlineAttribute = "5CA001";
    
    // ===== ISO 19157 품질요소 3: 위치 정확도 (Positional Accuracy) =====
    // 3단계: 지오메트리 검수 - 위치 정확도
    public const string ShortObject = "3SH001";           // 짧은 객체
    public const string SmallArea = "3SA001";             // 작은 면적
    public const string Sliver = "3SL001";                // 슬리버
    public const string Spike = "3SP001";                 // 스파이크
    public const string Undershoot = "3US001";            // 언더슛
    public const string Overshoot = "3OS001";             // 오버슛
    
    // 5단계: 공간 관계 검수 - 위치 정확도
    public const string PointSpacing = "5PS001";
    
    // ===== ISO 19157 품질요소 4: 시간 정확도 (Temporal Accuracy) =====
    // 4단계: 속성 관계 검수 - 시간 정확도 (향후 확장)
    public const string TemporalAccuracy = "4TM001";
    
    // ===== ISO 19157 품질요소 5: 주제 정확도 (Thematic Accuracy) =====
    // 4단계: 속성 관계 검수 - 주제 정확도
    public const string CodeListCheck = "4CD001";
    public const string RegexCheck = "4RG001";
    public const string RangeCheck = "4RN001";
    public const string ConditionalCheck = "4IF001";
    
    // 4단계: 속성 관계 검수 - 속성 일관성
    public const string NotNullAttribute = "4NO001";
    public const string NotZero = "4NZ001";
    public const string EqualsCheck = "4EQ001";
    public const string MultipleOf = "4MO001";
}
```

2. **CSV 로더 수정**
   - RuleId 컬럼 필수화
   - OriginalRuleId 컬럼 선택적 지원
   - RuleId 유효성 검증 로직 추가

3. **검수 프로세서 수정**
   - RuleId 기반 로깅 및 추적
   - RuleId 기반 오류 분류
   - RuleId 기반 통계 집계

## 6. 관리 도구 제안

### 6.1 규칙 관리 대시보드
- 규칙 목록 조회/검색
- 규칙 활성화/비활성화
- 규칙 변경 이력 조회
- 규칙 사용 통계

### 6.2 규칙 검증 도구
- RuleId 중복 검사
- RuleId 형식 검증
- 규칙 간 의존성 검사
- 누락된 규칙 검출

### 6.3 자동화 스크립트
- CSV 파일 RuleId 자동 생성
- 기존 RuleId → 새 RuleId 변환
- RuleId 문서 자동 생성

## 7. 예상 효과

### 7.1 관리 효율성
- 규칙 추적 용이: 고유 ID로 규칙 식별
- 변경 이력 관리: 규칙 변경 추적 가능
- 확장성 향상: 체계적 ID 체계로 확장 용이

### 7.2 개발 효율성
- 코드 가독성 향상: 명확한 규칙 ID
- 디버깅 용이: RuleId로 오류 추적
- 문서화 자동화: ID 기반 문서 생성

### 7.3 사용자 경험
- 오류 메시지 개선: RuleId 포함으로 명확한 오류 식별
- 통계 분석 향상: RuleId 기반 집계
- 보고서 품질 향상: 체계적 분류로 보고서 구조화

## 8. 실행 계획

### 8.1 우선순위
1. **High**: 4, 5단계 기존 RuleId 변환
2. **High**: 1, 2, 3단계 RuleId 추가
3. **Medium**: 코드리스트 및 기준 연계
4. **Medium**: 관리 도구 개발
5. **Low**: 자동화 스크립트 개발

### 8.2 일정
- **Week 1-2**: 설계 및 검토
- **Week 3-4**: Phase 1 (기존 규칙 변환)
- **Week 5-6**: Phase 2 (신규 규칙 추가)
- **Week 7**: Phase 3 (연계 작업)
- **Week 8**: 테스트 및 문서화

## 9. ISO 19157 품질요소별 RuleID 매핑 요약

### 9.1 완전성 (Completeness) - RuleID: `1*`

| RuleID 패턴 | 검수 항목 | 설명 |
|------------|----------|------|
| `1TB*` | 테이블 존재 | 필수 테이블/피처클래스 존재 여부 |
| `1FC*` | 피처클래스 존재 | 필수 피처클래스 존재 여부 |
| `1CR*` | 좌표계 지정 | 좌표계 메타데이터 지정 여부 |
| `1GT*` | 지오메트리 타입 | 지오메트리 타입 일치 여부 |

### 9.2 논리적 일관성 (Logical Consistency) - RuleID: `2*`, `3*`, `5*`

| RuleID 패턴 | 검수 항목 | 설명 |
|------------|----------|------|
| `2FD*` | 필드 정의 | 필드 존재 여부 |
| `2DT*` | 데이터 타입 | 데이터 타입 일치 |
| `2UK*` | 고유키 | 고유키 중복 검사 |
| `2FK*` | 외래키 | 외래키 참조 무결성 |
| `2NN*` | 필수값 | 필수값 NULL 검사 |
| `3VL*` | 지오메트리 유효성 | IsValid 검사 |
| `3NU*` | NULL 지오메트리 | NULL 검사 |
| `3EM*` | 빈 지오메트리 | IsEmpty 검사 |
| `3SM*` | 지오메트리 단순성 | IsSimple 검사 |
| `3DP*` | 객체 중복 | 객체 중복 검사 |
| `3OV*` | 객체 간 겹침 | 객체 간 겹침 검사 |
| `3SI*` | 자기교차 | 자체 꼬임 검사 |
| `3SO*` | 자기중첩 | 자기 중첩 검사 |
| `3HO*` | 홀 폴리곤 | 홀 폴리곤 오류 검사 |
| `5PI*` | 점-면 포함 | 점이 폴리곤 내부에 있는지 |
| `5LW*` | 선-면 포함 | 선이 폴리곤 내부에 있는지 |
| `5PW*` | 면-면 포함 | 폴리곤이 다른 폴리곤 내부에 있는지 |
| `5PN*` | 면-면 겹침 금지 | 폴리곤 간 겹침 금지 |
| `5LC*` | 선 연결성 | 선 연결성 검사 |
| `5CA*` | 중심선 속성 | 중심선 속성 불일치 검사 |

### 9.3 위치 정확도 (Positional Accuracy) - RuleID: `3*`, `5*`

| RuleID 패턴 | 검수 항목 | 설명 |
|------------|----------|------|
| `3SH*` | 짧은 객체 | 최소 선 길이 검사 |
| `3SA*` | 작은 면적 | 최소 면적 검사 |
| `3SL*` | 슬리버 | 슬리버 폴리곤 검사 |
| `3SP*` | 스파이크 | 스파이크 검사 |
| `3US*` | 언더슛 | 언더슛 검사 |
| `3OS*` | 오버슛 | 오버슛 검사 |
| `5PS*` | 점 간격 | 점 간 최소 거리 검사 |

### 9.4 시간 정확도 (Temporal Accuracy) - RuleID: `4TM*`

| RuleID 패턴 | 검수 항목 | 설명 |
|------------|----------|------|
| `4TM*` | 시간 정확도 | 시간 측정 정확도 (향후 확장) |

### 9.5 주제 정확도 (Thematic Accuracy) - RuleID: `4*`

| RuleID 패턴 | 검수 항목 | 설명 |
|------------|----------|------|
| `4CD*` | 코드리스트 | 코드값 유효성 검사 |
| `4RG*` | 정규식 | 형식 일치 검사 |
| `4RN*` | 범위 | 값 범위 검사 |
| `4IF*` | 조건부 | 조건부 속성 검사 |
| `4NO*` | 필수값 | 필수 속성 NULL 검사 |
| `4NZ*` | 0 금지 | 0 값 금지 검사 |
| `4EQ*` | 일치 | 값 일치 검사 |
| `4MO*` | 배수 | 배수 검사 |

## 10. 참고 자료

- ISO 19157:2013 Geographic information — Data quality
- ISO 19115-1:2014 Geographic information — Metadata — Part 1: Fundamentals
- ISO 19107:2003 Geographic information — Spatial schema
- 국가기본도 DB 검수 규칙서 v2.1.0

---

**작성일**: 2025-11-18  
**작성자**: SpatialCheckPro 개발팀  
**버전**: 2.0 (ISO 19157 품질요소 매핑 추가, 기본 검수 항목 추가)

