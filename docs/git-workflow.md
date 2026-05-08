# Git Workflow

# Plain Note Git 작업 규칙

## 1. 목적

이 문서는 Plain Note 프로젝트에서 Git과 GitHub를 사용하는 방식을 정리한다.

이 프로젝트는 혼자 개발하더라도 실제 현업 개발 흐름에 가깝게 진행한다.  
모든 작업은 Issue를 기준으로 시작하고, 별도 브랜치에서 작업한 뒤 Pull Request를 통해 `develop` 브랜치에 병합한다.

---

## 2. 기본 브랜치 구조

### main

`main` 브랜치는 배포 가능한 안정 버전만 유지한다.

- 직접 push하지 않는다.
- 충분히 검증된 develop 브랜치의 변경사항만 병합한다.
- 실제 배포 또는 릴리즈 기준 브랜치로 사용한다.

### develop

`develop` 브랜치는 개발 통합 브랜치이다.

- 기능 개발이 완료된 PR은 develop 브랜치로 병합한다.
- 다음 배포 후보가 되는 코드가 모이는 브랜치이다.
- 일반적인 개발 기준 브랜치로 사용한다.

### 작업 브랜치

각 작업은 별도의 브랜치에서 진행한다.

작업 브랜치는 Issue 하나를 해결하기 위한 단위로 만든다.

예시:

- `docs/3-project-docs`
- `chore/1-init-backend`
- `feature/4-note-create-api`
- `fix/8-note-update-error`

---

## 3. 작업 흐름

기본 작업 흐름은 다음과 같다.

1. GitHub Issue를 생성한다.
2. `develop` 브랜치를 최신 상태로 가져온다.
3. Issue 번호를 포함한 작업 브랜치를 만든다.
4. 작업을 진행한다.
5. 로컬에서 실행 또는 테스트를 확인한다.
6. 커밋한다.
7. 원격 저장소에 push한다.
8. Pull Request를 생성한다.
9. PR 내용을 확인한 뒤 `develop`에 squash merge한다.
10. 작업 브랜치를 삭제한다.

---

## 4. 명령어 흐름

### 4.1 작업 시작

```bash
git switch develop
git pull origin develop
git switch -c 브랜치이름
```

### 4.2 작업 후 커밋

```bash
git status
git add .
git commit -m "커밋 메시지"
```

### 4.3. 원격 저장소에 push

```bash
git push -u origin 브랜치이름
```

### 4.4. PR 병합 후 로컬 정리

```bash
git switch develop
git pull origin develop
git branch -d 브랜치이름
git fetch --prune
```

## 5. 브랜치 이름 규칙

브랜치 이름은 다음 형식을 사용한다.

```
작업종류/이슈번호-작업요약
```

## 6. 브랜치 prefix 규칙

### docs

문서 작업에 사용한다.

### chore

프로젝트 설정, 의존성 추가, 빌드 설정 등 기능과 직접 관련 없는 작업에 사용한다.

### feature

새로운 기능을 추가할 때 사용한다.

### fix

버그를 수정할 때 사용한다.

### refactor

기능 변화 없이 코드 구조를 개선할 때 사용한다.

### test

테스트 코드를 추가하거나 수정할 때 사용한다.

## 7. Issue 작성 규칙

Issue는 하나의 작업 단위로 작성한다.

좋은 Issue는 다음 조건을 가진다.

- 작업 목적이 명확하다.
- 구현할 내용이 체크리스트로 정리되어 있다.
- 완료 조건이 분명하다.
- 하나의 PR로 해결할 수 있다.

## 8. 커밋 메시지 규칙

커밋 메시지는 다음 형식을 사용한다.

```
type: 작업 내용
```

## 9. 커밋 type 규칙
- feat: 새로운 기능 추가
- fix: 버그 수정
- docs: 문서 수정
- chore: 설정, 빌드, 의존성, 초기 세팅 작업
- refactor: 기능 변화 없는 코드 구조 개선
- test: 테스트 코드 추가 또는 수정
- style: 코드 포맷팅 수정

## 10. Pull Request 규칙

PR은 작업 브랜치에서 develop 브랜치로 생성한다.

기본 방향은 다음과 같다.

```
base: develop
compare: 작업 브랜치
```

## 11. Merge 방식

PR은 기본적으로 Squash and merge를 사용한다.

이유는 다음과 같다.

- feature 브랜치의 여러 커밋을 하나로 정리할 수 있다.
- develop 브랜치의 커밋 히스토리를 깔끔하게 유지할 수 있다.
- PR 단위로 변경 내역을 추적하기 쉽다.

## 12. main 병합 규칙

`main` 브랜치에는 일반 기능 브랜치를 직접 병합하지 않는다.

기본 흐름은 다음과 같다.

```
feature 브랜치
→ develop
→ main
```

`main` 병합은 다음 조건을 만족할 때만 진행한다.

- 핵심 기능이 안정적으로 동작한다.
- build가 성공한다.
- 주요 API가 정상 동작한다.
- 배포 가능한 상태라고 판단된다.

## 13. 현재 개발 흐름

현재 Plain Note 프로젝트는 다음 순서로 개발한다.

1. Spring Boot 백엔드 초기 세팅
1. 프로젝트 기본 문서 작성
1. Note 도메인 기본 구조 작성
1. Note 생성 API 구현
1. Note 목록 조회 API 구현
1. Note 단건 조회 API 구현
1. Note 수정 API 구현
1. Note 삭제 API 구현
1. 공통 예외 처리 구조 작성
1. DB 연결
1. JPA 기반 Note CRUD 전환
1. 사용자 인증
1. 폴더 기능
1. 태그 기능
1. 검색 기능
1. 프론트엔드 구현

## 14.  기본 원칙
- main에는 직접 push하지 않는다.
- 하나의 Issue는 하나의 PR로 해결한다.
- 브랜치는 Issue 번호를 포함해 만든다.
- PR은 develop을 대상으로 만든다.
- 병합은 Squash and merge를 사용한다.
- 작업 전에는 항상 develop을 최신화한다.
- 너무 큰 작업은 여러 Issue로 나눈다.