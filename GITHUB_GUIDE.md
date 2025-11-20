# GitHub 연결 및 커밋 가이드

## 1. 현재 변경사항 확인
```powershell
git status
```

## 2. 변경사항을 스테이징 영역에 추가
```powershell
# 모든 변경사항 추가
git add .

# 또는 특정 파일만 추가
git add SpatialCheckPro.GUI/ViewModels/ValidationSettingsViewModel.cs
git add SpatialCheckPro.GUI/MainWindow.xaml.cs
```

## 3. 커밋 생성
```powershell
git commit -m "feat: MVVM 패턴 완성 - ValidationSettingsViewModel 통합

- ValidationSettingsViewModel에 모든 검수 설정 데이터 집중
- MainWindow의 internal 필드 제거 및 ViewModel 사용
- ValidationSettingsView의 Reflection 기반 코드 제거
- DI 컨테이너에 ValidationSettingsViewModel 등록
- 빌드 오류 전수 해결 (0 errors)"
```

## 4. GitHub 원격 저장소 연결 (처음 한 번만)
```powershell
# GitHub에서 새 저장소를 만든 후, 아래 명령 실행
git remote add origin https://github.com/wgsystem-1/SpatialCheckPro2.git

# 이미 연결되어 있는지 확인
git remote -v
```

## 5. GitHub에 푸시
```powershell
# 첫 푸시 (main 브랜치)
git push -u origin main

# 이후 푸시
git push
```

## 6. 브랜치 작업 (선택사항)
```powershell
# 새 브랜치 생성 및 전환
git checkout -b feature/mvvm-refactoring

# 브랜치 푸시
git push -u origin feature/mvvm-refactoring
```

## 주의사항

### .gitignore 파일 확인
다음 내용이 `.gitignore`에 포함되어 있는지 확인하세요:

```gitignore
# Build results
[Dd]ebug/
[Dd]ebugPublic/
[Rr]elease/
[Rr]eleases/
x64/
x86/
[Ww][Ii][Nn]32/
[Aa][Rr][Mm]/
[Aa][Rr][Mm]64/
bld/
[Bb]in/
[Oo]bj/
[Ll]og/
[Ll]ogs/

# Visual Studio cache/options
.vs/
*.suo
*.user
*.userosscache
*.sln.docstates

# NuGet Packages
*.nupkg
**/packages/*
!**/packages/build/

# User-specific files
*.rsuser
*.suo
*.user
*.userosscache
*.sln.docstates

# Build logs
*.log
build_*.txt
```

## 커밋 메시지 컨벤션

좋은 커밋 메시지 작성 예시:
- `feat:` 새로운 기능 추가
- `fix:` 버그 수정
- `refactor:` 코드 리팩토링
- `docs:` 문서 수정
- `test:` 테스트 코드 추가/수정
- `chore:` 빌드 설정, 패키지 등

예시:
```
feat: ValidationSettingsViewModel 구현

- 검수 설정 데이터를 ViewModel로 중앙화
- MVVM 패턴 준수를 위한 리팩토링
```
