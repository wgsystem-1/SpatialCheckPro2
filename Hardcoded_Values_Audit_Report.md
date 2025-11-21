# 하드코딩된 설정 값 감사 보고서 (Audit Report on Hardcoded Configuration Values)

**날짜:** 2025-11-21
**대상 프로젝트:** SpatialCheckPro2

이 보고서는 프로젝트 내 소스 코드(`.cs`) 및 설정 파일에서 발견된 하드코딩된 경로, 설정 값, 그리고 잠재적인 문제점에 대한 전수 조사 결과입니다.

## 1. 하드코딩된 파일 경로 (Hardcoded File Paths)

소스 코드 내에 직접 명시된 절대 경로는 배포 환경이나 다른 개발 환경에서 오류를 일으킬 수 있는 주요 원인입니다.

### 1.1. 설정 파일 폴더 경로 (Fallback)
*   **파일:** `SpatialCheckPro.GUI\Views\ValidationResultView.xaml.cs`
*   **위치:** 463행 (GetDefaultConfigDirectory 메서드)
*   **코드:** `Path.Combine(@"G:\SpatialCheckPro\SpatialCheckPro\Config")`
*   **설명:** 설정 파일을 찾지 못했을 때 사용하는 대체(Fallback) 경로 중 하나로 개발자의 로컬 경로(`G:\`)가 하드코딩되어 있습니다.
*   **영향:** 배포 환경에서 해당 드라이브나 경로가 없을 경우, 앞선 경로 탐색이 실패하면 예외가 발생할 수 있습니다.

### 1.2. PostgreSQL 설치 경로
*   **파일:** `SpatialCheckPro.GUI\Services\GdalInitializationService.cs`
*   **위치:** 263행
*   **코드:** `var postgresPath = @"C:\Program Files\PostgreSQL";`
*   **설명:** 시스템 PATH 환경변수에서 충돌을 방지하기 위해 PostgreSQL의 PROJ 라이브러리 경로를 제거하는 로직에 사용됩니다.
*   **영향:** PostgreSQL이 다른 경로(예: D 드라이브)에 설치된 경우, 충돌 방지 로직이 작동하지 않을 수 있습니다.

### 1.3. 시스템 폰트 경로
*   **파일:** `SpatialCheckPro\Services\PdfReportService.cs`
*   **위치:** 674-677행 (GetKoreanFontPath 메서드)
*   **코드:**
    *   `@"C:\Windows\Fonts\malgun.ttf"`
    *   `@"C:\Windows\Fonts\gulim.ttc"`
    *   등 Windows 기본 폰트 경로
*   **설명:** PDF 보고서 생성 시 한글 출력을 위해 Windows 시스템 폰트 경로를 직접 참조합니다.
*   **영향:** Windows가 아닌 OS나 폰트 설치 경로가 다른 환경에서는 한글이 깨지거나 보고서 생성이 실패할 수 있습니다.

### 1.4. 벤치마크 샘플 경로 (주석)
*   **파일:** `SpatialCheckPro.GUI\Views\ValidationSettingsView.xaml.cs`
*   **위치:** 1249행
*   **코드:** `// var sampleGdbPath = @"G:\SpatialCheckPro\sample.gdb";`
*   **설명:** 주석 처리된 벤치마크 테스트 코드 내에 하드코딩된 경로가 존재합니다.
*   **영향:** 현재 주석 처리되어 있어 실행 시 영향은 없으나, 추후 주석 해제 시 수정이 필요합니다.

---

## 2. 하드코딩된 설정 값 (Hardcoded Configuration Values)

코드 내에 상수로 박혀 있어, 변경하려면 재컴파일이 필요한 값들입니다.

### 2.1. 초기 예측 데이터 및 이력 파일명
*   **파일:** `SpatialCheckPro.GUI\Models\ValidationTimePredictor.cs`
*   **위치:**
    *   60행: `"validation_history.json"` (이력 파일명)
    *   99행 이하: `InitializeWithDefaultData` 메서드 내의 초기 데이터 값들
*   **설명:** 검수 시간 예측을 위한 초기 데이터와 이력 저장 파일명이 코드에 고정되어 있습니다.
*   **영향:** 초기 설치 시 예측 시간이 실제 환경과 맞지 않을 수 있으며, 이력 파일의 위치나 이름을 변경하기 어렵습니다.

### 2.2. QC 결과 폴더 및 명명 규칙
*   **파일:** `SpatialCheckPro\Services\QcErrorsPathManager.cs`
*   **위치:** 15-25행
*   **코드:**
    *   `QC_ERRORS_FOLDER = "QC_errors"`
    *   `QC_SUFFIX = "_QC"`
    *   `ExcludedPatterns` 배열 내용
*   **설명:** 오류 결과를 저장할 폴더명(`QC_errors`)과 결과 GDB의 접미사(`_QC`), 그리고 탐색 제외 패턴이 상수로 정의되어 있습니다.
*   **영향:** 사용자가 결과 폴더명을 변경하고 싶거나 제외 규칙을 수정하려면 소스 코드를 수정해야 합니다.

### 2.3. GDAL 디버그 로그 파일명
*   **파일:** `SpatialCheckPro.GUI\Services\GdalInitializationService.cs`
*   **위치:** 139행
*   **코드:** `"gdal_debug.log"`
*   **설명:** GDAL 라이브러리의 디버그 로그 파일명이 고정되어 있습니다.

---

## 3. appsettings.json 설정 (Configuration Defaults)

`appsettings.json` 파일에는 다음과 같은 주요 설정값들이 정의되어 있습니다. 이는 하드코딩이라기보다 **기본 설정값**으로 간주되지만, 프로그램 동작에 중요한 영향을 미칩니다.

*   **FileProcessing:**
    *   `MaxFileSizeBytes`: 약 1.8GB (대용량 파일 기준)
    *   `ChunkSizeBytes`: 100MB
*   **Performance:**
    *   `MaxMemoryUsageMB`: 2048 (2GB)
    *   `HighPerformanceModeFeatureThreshold`: 400,000 (객체 수 기준)

---

## 4. 권장 사항 (Recommendations)

1.  **경로 설정의 외부화:** `ValidationResultView.xaml.cs` 및 `GdalInitializationService.cs`에 있는 절대 경로들을 `appsettings.json`으로 이동하여 관리자가 수정할 수 있도록 변경하는 것이 좋습니다.
2.  **폰트 처리 개선:** `PdfReportService.cs`에서 폰트 경로를 설정 파일에서 읽어오거나, 폰트가 없을 경우의 대체 로직(Fallback)을 강화해야 합니다.
3.  **상수 값의 설정화:** `QcErrorsPathManager.cs`의 폴더명이나 접미사 등도 설정 파일로 관리하면 유지보수성이 향상됩니다.
