# SpatialCheckPro 검수 설정 가이드

이 디렉토리는 SpatialCheckPro의 검수 규칙을 정의하는 설정 파일(`csv`)들을 포함하고 있습니다.
각 파일은 검수 단계(1~5단계)별 규칙과 기준 정보를 담고 있으며, 프로그램 실행 시 이 파일들을 로드하여 동적으로 검수를 수행합니다.

## 설정 파일 목록

| 파일명 | 단계 | 설명 |
| :--- | :---: | :--- |
| **`1_table_check.csv`** | 1 | **테이블 검수**: 필수 레이어 존재 여부, 지오메트리 타입, 좌표계(CRS) 정의 |
| **`2_schema_check.csv`** | 2 | **스키마 검수**: 필드 구조, 데이터 타입, 길이, 제약조건(PK, FK, UK, Not Null) 정의 |
| **`3_geometry_check.csv`** | 3 | **지오메트리 검수**: 객체 단위의 공간적 오류(중복, 꼬임, 미세 객체 등) 검사 활성화 여부 |
| **`geometry_criteria.csv`** | 3+ | **지오메트리 기준값**: 지오메트리 검수 시 사용되는 임계값(Tolerance) 및 기준값 정의 |
| **`4_attribute_check.csv`** | 4 | **속성 검수**: 속성값의 유효성, 범위, 코드리스트 준수, 정규식 패턴 등 검사 규칙 |
| **`codelist.csv`** | 4+ | **코드리스트**: 속성 검수에서 참조하는 공통 코드 정의서 |
| **`5_relation_check.csv`** | 5 | **관계 검수**: 레이어 간 위상 관계(포함, 교차, 이격 등) 및 공간 연관성 검사 규칙 |

---

## 공통 설정 규칙

### 규칙 비활성화 (주석 처리)
`RuleId` 앞에 `#` 문자를 추가하면 해당 규칙은 **주석 처리되어 검수에서 제외**됩니다.
이는 `Enabled` 컬럼을 `N`으로 설정하는 것보다 더 우선적으로 적용되며, 임시로 규칙을 끌 때 유용합니다.

*   **적용 대상**: 4단계(속성 검수), 5단계(관계 검수)
*   **예시**: `#ATTR_CODE_BULD` (속성 검수), `#건물_도로경계면_침범` (관계 검수)

---

## 상세 파일 구조

### 1. 테이블 검수 (`1_table_check.csv`)
File Geodatabase(GDB) 내에 반드시 존재해야 하는 테이블(레이어) 목록과 기본 속성을 정의합니다.

*   **TableId**: 테이블(Feature Class) 물리적 이름 (예: `tn_buld`)
*   **TableName**: 테이블 논리적 이름/한글명 (예: `건물`)
*   **GeometryType**: 기대하는 공간 타입 (`POLYGON`, `LINESTRING`, `POINT` 등)
*   **CRS**: 좌표계 정의 (예: `EPSG:5179`)

---

### 2. 스키마 검수 (`2_schema_check.csv`)
각 테이블의 컬럼(필드) 구조와 무결성 제약조건을 정의합니다.

*   **TableId**: 대상 테이블 ID
*   **FieldName**: 필드 영문명
*   **FieldAlias**: 필드 한글명/별칭
*   **DataType**: 데이터 타입 (`String`, `Integer`, `NUMERIC`, `Date` 등)
*   **Length**: 필드 길이 (문자열 길이 또는 `전체자리수,소수점자리수`)
*   **UK**: Unique Key 여부 (`Y` 또는 공백)
*   **FK**: Foreign Key 여부 (`FK` 또는 공백)
*   **NN**: Not Null 여부 (`Y` 또는 공백)
*   **RefTable**: FK인 경우 참조할 부모 테이블 ID
*   **RefColumn**: FK인 경우 참조할 부모 컬럼명

---

### 3. 지오메트리 검수 (`3_geometry_check.csv`)
각 테이블별로 수행할 지오메트리 검사 항목을 On/Off(`Y`/`N`) 합니다.

*   **TableId**: 대상 테이블 ID
*   **TableName**: 테이블 한글명
*   **GeometryType**: 지오메트리 타입
*   **검사 항목 컬럼들**:
    *   `객체중복`: 동일한 좌표를 가진 객체 검출
    *   `객체간겹침`: 서로 겹치는 객체 검출
    *   `자체꼬임`: Self-Intersection 검출
    *   `슬리버`: 매우 얇거나 작은 면적의 슬리버 폴리곤 검출
    *   `짧은객체`: 기준 길이 미만의 선 검출
    *   `작은면적객체`: 기준 면적 미만의 폴리곤 검출
    *   `홀 폴리곤 오류`: 폴리곤 내부 홀(Hole)의 위상 오류 검출
    *   `최소정점개수`: 구성 정점 수가 부족한 객체 검출
    *   `스파이크`: 급격하게 꺾이는 스파이크 형상 검출
    *   `자기중첩`: 링이나 선분이 자기 자신과 겹치는 경우
    *   `언더슛/오버슛`: 선 연결성 오류 (네트워크 데이터용)

#### 지오메트리 기준값 (`geometry_criteria.csv`)
3단계 검수에서 사용되는 전역 기준값입니다.

*   **항목명**: 기준 항목 (예: `최소선길이`, `겹침허용면적`)
*   **값**: 설정값 (예: `0.01`)
*   **단위**: 단위 설명 (예: `미터`, `제곱미터`)
*   **설명**: 항목에 대한 설명

---

### 4. 속성 검수 (`4_attribute_check.csv`)
필드 값의 논리적 유효성을 검사하는 규칙입니다.

*   **RuleId**: 규칙 식별자
*   **Enabled**: 활성화 여부 (`Y`/`N`)
*   **TableId**: 대상 테이블 ID (와일드카드 `*` 사용 가능)
*   **TableName**: 테이블명
*   **FieldName**: 검사할 필드명
*   **CheckType**: 검사 유형
    *   `CodeList`: `codelist.csv`에 정의된 코드값인지 확인
    *   `Range`: 숫자 범위 검사 (`min..max`)
    *   `Regex`: 정규식 패턴 일치 여부
    *   `NotZero`: 0이 아닌 값이어야 함
    *   `NumericEquals`: 특정 숫자와 일치해야 함
    *   `MultipleOf`: 특정 수의 배수여야 함
    *   `KoreanTypo`: 한글 오타(자모 분리 등) 검사
    *   `IfCodeThenMultipleOf`: 조건부 배수 검사
*   **Parameters**: 검사 유형에 따른 파라미터 (코드셋ID, 범위값, 정규식 등)

#### 코드리스트 (`codelist.csv`)
`CodeList` 타입 검사에서 참조하는 코드 정의 파일입니다.

*   **CodeSetId**: 코드 그룹 ID (예: `건물구분V6`)
*   **CodeValue**: 유효한 코드값 (예: `BD001`)
*   **Label**: 코드 설명 (예: `일반주택`)

---

### 5. 관계 검수 (`5_relation_check.csv`)
두 레이어(Feature Class) 간의 공간적 위상 관계를 검사합니다.

*   **RuleId**: 규칙 식별자
*   **Enabled**: 활성화 여부 (`Y`/`N`)
*   **CaseType**: 관계 검사 유형
    *   `PolygonNotOverlap`: 폴리곤 간 겹침 금지
    *   `PolygonNotIntersectLine`: 폴리곤과 선의 교차 금지
    *   `LineWithinPolygon`: 선이 폴리곤 내부에 있어야 함
    *   `PolygonWithinPolygon`: 폴리곤이 다른 폴리곤 내부에 있어야 함
    *   `LineConnectivity`: 선의 연결성 검사
    *   `AttributeSpatialMismatch`: 공간 중첩 시 속성 일치 여부 검사
    *   `CenterlineAttributeMismatch`: 연결된 중심선의 속성 연속성 검사
*   **MainTableId**: 주 검사 대상 테이블
*   **RelatedTableId**: 관계 비교 대상 테이블
*   **FieldFilter**: 검사 시 적용할 속성 필터 또는 매핑 정보
*   **Tolerance**: 공간 연산 허용 오차 (미터 단위)
