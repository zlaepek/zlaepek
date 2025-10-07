# GitHub 프로필 자동 업데이트 시스템 설명

## 개요
GitHub Actions를 활용해 매일 자동으로 프로필을 업데이트하는 시스템입니다. 
실시간 통계, 언어 트렌드 차트, 최근 활동 등을 자동으로 분석하고 README에 반영합니다.

## 전체 구조

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│  GitHub Actions │───▶│  데이터 수집     │───▶│  README 업데이트 │
│  (스케줄러)      │    │  & 분석        │    │  & 커밋         │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

## 2개의 워크플로우

### 1. generate-charts.yml (언어 트렌드 차트 생성)
**실행 시점:** 매주 월요일 오전 9시
**주요 기능:**
- GitHub API로 리포지토리 언어 통계 수집
- 2020년부터 현재까지 월별 언어 사용 트렌드 분석
- 기술 스택별 분류 및 시각화
- PNG 차트 이미지 생성 및 assets 폴더 저장

```python
# Python 스크립트가 실행됨
1. GitHub API 호출 → 모든 리포지토리 정보 가져오기
2. 각 리포의 언어별 바이트 수 → 줄 수로 변환 (50바이트 ≈ 1줄)
3. 연도별 JSON 파일로 저장 (monthly_data_YYYY.json)
4. matplotlib로 2개 차트 생성:
   - language_trend_chart.png (2025년 월별 트렌드)
   - language_yearly_chart.png (연도별 기술스택 분포)
5. 분야별 색상 통일:
   - 게임개발(C#, Unity, C++): 파란색 계열
   - 웹개발(JS, TS, HTML, CSS): 노란색/녹색 계열  
   - 기타(Python, Java): 보라색 계열
```

### 2. update-profile.yml (프로필 통계 업데이트) 
**실행 시점:** 매일 오전 9시 (한국시간)
**주요 기능:**
- 현재 스킬 레벨 계산 및 업데이트
- 기술 스택 변화 추이 테이블 생성
- 최근 프로젝트별 활동 수집

```javascript
// Node.js 스크립트가 실행됨
1. GitHub API로 사용자 리포지토리 목록 가져오기
2. 각 리포의 언어별 바이트 수 → 줄 수 변환 → 스킬 레벨 계산
   레벨 = min(99, floor(log2(줄수/100 + 커밋수*10 + 1) * 8))
3. 최근 30개 이벤트에서 프로젝트별 커밋 활동 추출
4. README의 3개 섹션 업데이트:
   - <!-- SKILL_START --> 스킬 뱃지 (레벨 표시 제거)
   - <!-- TREND_START --> 기술 스택 변화 추이
   - <!-- ACTIVITY_START --> 최근 프로젝트별 활동
5. 업데이트 시간 갱신
```

## 데이터 수집 방식

### GitHub API 호출 구조:
```
GET /users/{username}/repos          → 리포지토리 목록
GET /repos/{owner}/{repo}/languages  → 언어별 바이트 수
GET /users/{username}/events         → 최근 활동 (커밋, 이슈 등)
```

### 스킬 레벨 계산 로직:
```
활동점수 = (언어 줄수 ÷ 100) + (커밋수 × 10)
레벨 = min(99, floor(log2(활동점수 + 1) × 8))

예시:
- JavaScript 5000줄 + 20커밋 = 50 + 200 = 250점 → 레벨 65
- Python 1000줄 + 5커밋 = 10 + 50 = 60점 → 레벨 49
```

### 기술 스택 분류 시스템:
```
게임개발: C#, Unity, C++
웹개발: JavaScript, TypeScript, HTML, CSS, React, Vue
백엔드: Python, Java, Node.js
기타: 위에 해당하지 않는 모든 언어
```

## README 업데이트 방식

특별한 주석을 이용해 특정 섹션만 교체:

```markdown
<!-- SKILL_START -->
현재 스킬 테이블이 자동으로 교체됨
<!-- SKILL_END -->

<!-- TREND_START -->
기술 스택 변화 추이 테이블이 자동으로 업데이트됨
<!-- TREND_END -->

<!-- ACTIVITY_START -->
최근 프로젝트별 활동이 자동으로 업데이트됨
<!-- ACTIVITY_END -->

<!-- TREND_CHART --> / <!-- YEARLY_CHART -->
차트 이미지 링크가 자동으로 업데이트됨
```

정규표현식으로 해당 구간을 찾아서 새로운 내용으로 교체합니다.

## 커밋 및 푸시 과정

1. 변경사항 스테이징: `git add README.md assets/`
2. 변경사항 체크: `git diff --staged --quiet`  
3. 변경사항 있으면 커밋: `git commit -m "메시지"`
4. 원격 저장소 푸시: `git push`

## 권한 설정

```yaml
permissions:
  contents: write  # README 파일 수정 및 커밋 권한
```

GitHub Actions가 자동으로 GITHUB_TOKEN을 생성해서 인증에 사용합니다.

## 차트 디자인 특징

### 2025 Language Trend Analysis
- **표시 기간**: 2025년 데이터만 필터링
- **분야별 그룹화**: 🎮게임개발(사각형), 🌐웹개발(삼각형), ⚙️기타(원형)
- **색상 체계**: 분야별 일관된 색상 사용
- **모던 스타일**: GitHub 다크테마, 개선된 폰트와 그리드

### Tech Stack Distribution by Year  
- **퍼센트 스택바**: 연도별 기술스택 비율 표시
- **3개 카테고리**: Game Dev(파란색), Web Dev(노란색), Others(보라색)
- **실제 데이터**: 샘플 데이터 사용하지 않음

## 실패 처리

- API 호출 실패시: 에러 로그 출력 후 스킵
- 데이터 없을시: 차트 생성 건너뛰기  
- ShaderLab 등 자동생성 언어 필터링
- Git 충돌 방지: `git pull origin main` 먼저 실행

## 개선된 기능들

1. **Weekly Report 제거**: 중복 정보 간소화
2. **프로젝트별 활동**: Recent Commits → Recent Projects로 변경
3. **레벨 표시 간소화**: Lv80 → Expert로 텍스트만 표시
4. **한글 폰트 문제 해결**: 차트 라벨을 영어로 변경
5. **연도별 데이터 보관**: monthly_data_YYYY.json으로 이력 관리

## 장점

1. **완전 자동화**: 손수 업데이트할 필요 없음
2. **실시간 반영**: 개발 활동이 바로 프로필에 표시
3. **비용 무료**: GitHub Actions 무료 사용량 내에서 실행
4. **시각적**: 모던한 차트와 뱃지로 한눈에 파악 가능
5. **확장성**: 새로운 통계나 기능 쉽게 추가 가능
6. **정확성**: 실제 코드 줄 수 기반 계산, 샘플 데이터 없음

이 시스템을 통해 개발자의 실제 활동이 자동으로 프로필에 반영되어, 
방문자가 현재 어떤 기술에 집중하고 있는지 쉽게 파악할 수 있습니다.