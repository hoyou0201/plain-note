# Plain Note API Spec

## 1. 개요

이 문서는 Plain Note 백엔드 API 명세를 정리한다.

Plain Note는 노션과 옵시디언이 어렵게 느껴지는 사용자를 위한 가벼운 클라우드 메모앱이다.

초기 MVP에서는 다음 기능을 제공한다.

- Health Check
- 메모 생성 / 조회 / 수정 / 삭제
- 폴더 생성 / 조회 / 수정 / 삭제
- 태그 조회
- 검색
- 사용자 회원가입 / 로그인

---

## 2. API 기본 규칙

### 2.1 Base URL

로컬 개발 환경 기준 Base URL은 다음과 같다.

`http://localhost:8080`

### 2.2 API Prefix

모든 API는 `/api` prefix를 사용한다.

예시:

- `GET /api/health`
- `GET /api/notes`
- `POST /api/notes`

### 2.3 Content-Type

요청과 응답은 기본적으로 JSON을 사용한다.

`Content-Type: application/json`

---

## 3. 공통 응답 형식

초기 개발에서는 단순 응답을 허용한다.

다만 추후 API 응답 형식을 통일하기 위해 다음 구조를 고려한다.

### 3.1 성공 응답 예시

| 필드 | 설명 |
|---|---|
| data | 실제 응답 데이터 |

예시:

| 필드 | 값 |
|---|---|
| data.id | 1 |
| data.title | 첫 메모 |

### 3.2 에러 응답 예시

| 필드 | 설명 |
|---|---|
| code | 에러 코드 |
| message | 사용자에게 보여줄 수 있는 에러 메시지 |

예시:

| 필드 | 값 |
|---|---|
| code | NOTE_NOT_FOUND |
| message | 메모를 찾을 수 없습니다. |

### 3.3 초기 구현 방침

초기 Note CRUD 구현 단계에서는 응답 wrapper 없이 DTO를 바로 반환할 수 있다.

예시:

| 필드 | 값 |
|---|---|
| id | 1 |
| title | 첫 메모 |
| content | 내용입니다. |

추후 공통 응답 구조가 필요해지면 `ApiResponse<T>` 형태로 통일한다.

---

## 4. 공통 에러 코드

| HTTP Status | Code | Message |
|---|---|---|
| 400 | INVALID_REQUEST | 잘못된 요청입니다. |
| 401 | UNAUTHORIZED | 인증이 필요합니다. |
| 403 | FORBIDDEN | 접근 권한이 없습니다. |
| 404 | NOT_FOUND | 요청한 리소스를 찾을 수 없습니다. |
| 404 | NOTE_NOT_FOUND | 메모를 찾을 수 없습니다. |
| 404 | FOLDER_NOT_FOUND | 폴더를 찾을 수 없습니다. |
| 409 | DUPLICATE_EMAIL | 이미 사용 중인 이메일입니다. |
| 500 | INTERNAL_SERVER_ERROR | 서버 내부 오류가 발생했습니다. |

---

## 5. Health API

서버 상태를 확인하기 위한 API이다.

### 5.1 Health Check

`GET /api/health`

#### Response

`OK`

#### 설명

- 서버가 정상 실행 중인지 확인한다.
- 로컬 개발 환경이나 배포 환경에서 상태 확인용으로 사용한다.

#### 현재 구현 상태

| 항목 | 상태 |
|---|---|
| API 구현 | 완료 |
| 로컬 실행 확인 | 완료 |
| 응답 확인 | 완료 |

---

## 6. Auth API

사용자 회원가입과 로그인을 위한 API이다.

초기 Note CRUD 구현 단계에서는 인증을 나중으로 미룰 수 있다.

다만 최종 MVP에서는 사용자별 메모 접근을 위해 인증이 필요하다.

### 6.1 회원가입

`POST /api/auth/signup`

#### Request Body

| 필드 | 타입 | 필수 | 설명 |
|---|---|---|---|
| email | String | O | 사용자 이메일 |
| password | String | O | 사용자 비밀번호 |
| nickname | String | O | 사용자 닉네임 |

#### Request 예시

| 필드 | 값 |
|---|---|
| email | user@example.com |
| password | password1234 |
| nickname | 진서 |

#### Response

| 필드 | 타입 | 설명 |
|---|---|---|
| id | Long | 사용자 id |
| email | String | 사용자 이메일 |
| nickname | String | 사용자 닉네임 |

#### Validation

| 필드 | 조건 |
|---|---|
| email | 필수, 이메일 형식 |
| password | 필수, 최소 8자 이상 |
| nickname | 필수, 1자 이상 100자 이하 |

#### Error

| 상황 | HTTP Status | Code |
|---|---|---|
| 이메일 중복 | 409 | DUPLICATE_EMAIL |
| 입력값 오류 | 400 | INVALID_REQUEST |

---

### 6.2 로그인

`POST /api/auth/login`

#### Request Body

| 필드 | 타입 | 필수 | 설명 |
|---|---|---|---|
| email | String | O | 사용자 이메일 |
| password | String | O | 사용자 비밀번호 |

#### Request 예시

| 필드 | 값 |
|---|---|
| email | user@example.com |
| password | password1234 |

#### Response

| 필드 | 타입 | 설명 |
|---|---|---|
| accessToken | String | JWT Access Token |
| tokenType | String | Bearer |

#### Error

| 상황 | HTTP Status | Code |
|---|---|---|
| 이메일 또는 비밀번호 불일치 | 401 | UNAUTHORIZED |
| 입력값 오류 | 400 | INVALID_REQUEST |

---

### 6.3 내 정보 조회

`GET /api/me`

#### Header

| 이름 | 값 |
|---|---|
| Authorization | Bearer {accessToken} |

#### Response

| 필드 | 타입 | 설명 |
|---|---|---|
| id | Long | 사용자 id |
| email | String | 사용자 이메일 |
| nickname | String | 사용자 닉네임 |

---

## 7. Notes API

메모 생성, 조회, 수정, 삭제를 담당한다.

초기 개발에서는 인증 없이 구현할 수 있다.

인증 도입 후에는 반드시 로그인한 사용자의 메모만 조회되도록 제한한다.

### 7.1 메모 생성

`POST /api/notes`

#### Request Body

| 필드 | 타입 | 필수 | 설명 |
|---|---|---|---|
| title | String | O | 메모 제목 |
| content | String | O | 메모 본문 |

#### Request 예시

| 필드 | 값 |
|---|---|
| title | 첫 메모 |
| content | Plain Note 첫 번째 메모입니다. |

#### Response

| 필드 | 타입 | 설명 |
|---|---|---|
| id | Long | 메모 id |
| title | String | 메모 제목 |
| content | String | 메모 본문 |
| favorite | Boolean | 즐겨찾기 여부 |
| createdAt | LocalDateTime | 생성일시 |
| updatedAt | LocalDateTime | 수정일시 |

#### Validation

| 필드 | 조건 |
|---|---|
| title | 필수, 1자 이상 200자 이하 |
| content | 필수, 빈 문자열 허용 여부는 정책으로 결정 |

#### Error

| 상황 | HTTP Status | Code |
|---|---|---|
| 제목이 비어 있음 | 400 | INVALID_REQUEST |
| 본문이 null | 400 | INVALID_REQUEST |

---

### 7.2 메모 목록 조회

`GET /api/notes`

#### Query Parameters

| 이름 | 필수 | 설명 |
|---|---|---|
| folderId | 선택 | 특정 폴더의 메모만 조회 |
| tag | 선택 | 특정 태그가 연결된 메모만 조회 |
| favorite | 선택 | 즐겨찾기 여부로 필터링 |
| page | 선택 | 페이지 번호 |
| size | 선택 | 페이지 크기 |

#### Request 예시

- `GET /api/notes`
- `GET /api/notes?folderId=1`
- `GET /api/notes?tag=study`

#### Response

| 필드 | 타입 | 설명 |
|---|---|---|
| id | Long | 메모 id |
| title | String | 메모 제목 |
| contentPreview | String | 본문 일부 미리보기 |
| favorite | Boolean | 즐겨찾기 여부 |
| createdAt | LocalDateTime | 생성일시 |
| updatedAt | LocalDateTime | 수정일시 |

#### 설명

- 기본 정렬은 `updatedAt DESC`로 한다.
- 삭제된 메모는 목록에서 제외한다.
- 인증 적용 후에는 로그인한 사용자의 메모만 반환한다.

---

### 7.3 메모 단건 조회

`GET /api/notes/{noteId}`

#### Path Variable

| 이름 | 설명 |
|---|---|
| noteId | 조회할 메모 id |

#### Response

| 필드 | 타입 | 설명 |
|---|---|---|
| id | Long | 메모 id |
| title | String | 메모 제목 |
| content | String | 메모 본문 |
| favorite | Boolean | 즐겨찾기 여부 |
| folderId | Long | 폴더 id |
| tags | List<String> | 태그 목록 |
| createdAt | LocalDateTime | 생성일시 |
| updatedAt | LocalDateTime | 수정일시 |

#### Error

| 상황 | HTTP Status | Code |
|---|---|---|
| 메모가 존재하지 않음 | 404 | NOTE_NOT_FOUND |
| 다른 사용자의 메모 접근 | 403 | FORBIDDEN |

---

### 7.4 메모 수정

`PATCH /api/notes/{noteId}`

#### Request Body

| 필드 | 타입 | 필수 | 설명 |
|---|---|---|---|
| title | String | 선택 | 수정할 제목 |
| content | String | 선택 | 수정할 본문 |
| folderId | Long | 선택 | 이동할 폴더 id |
| favorite | Boolean | 선택 | 즐겨찾기 여부 |

#### Request 예시

| 필드 | 값 |
|---|---|
| title | 수정된 제목 |
| content | 수정된 메모 내용입니다. |
| folderId | 1 |
| favorite | true |

#### Response

| 필드 | 타입 | 설명 |
|---|---|---|
| id | Long | 메모 id |
| title | String | 수정된 제목 |
| content | String | 수정된 본문 |
| folderId | Long | 폴더 id |
| favorite | Boolean | 즐겨찾기 여부 |
| createdAt | LocalDateTime | 생성일시 |
| updatedAt | LocalDateTime | 수정일시 |

#### 설명

- 제목 또는 본문을 수정할 수 있다.
- 폴더 이동도 이 API에서 처리할 수 있다.
- 수정 시 `updatedAt`이 갱신된다.

#### Error

| 상황 | HTTP Status | Code |
|---|---|---|
| 메모가 존재하지 않음 | 404 | NOTE_NOT_FOUND |
| 폴더가 존재하지 않음 | 404 | FOLDER_NOT_FOUND |
| 제목이 비어 있음 | 400 | INVALID_REQUEST |

---

### 7.5 메모 삭제

`DELETE /api/notes/{noteId}`

#### Response

`204 No Content`

#### 설명

- 초기 구현에서는 hard delete를 사용할 수 있다.
- 추후 휴지통 기능이 필요해지면 `deletedAt`을 사용하는 soft delete로 변경한다.

#### Error

| 상황 | HTTP Status | Code |
|---|---|---|
| 메모가 존재하지 않음 | 404 | NOTE_NOT_FOUND |
| 다른 사용자의 메모 삭제 시도 | 403 | FORBIDDEN |

---

### 7.6 메모 즐겨찾기 변경

`PATCH /api/notes/{noteId}/favorite`

#### Request Body

| 필드 | 타입 | 필수 | 설명 |
|---|---|---|---|
| favorite | Boolean | O | 즐겨찾기 여부 |

#### Response

| 필드 | 타입 | 설명 |
|---|---|---|
| id | Long | 메모 id |
| favorite | Boolean | 즐겨찾기 여부 |

#### 설명

- 즐겨찾기 여부를 변경한다.
- 초기 MVP에서 제외해도 된다.

---

## 8. Folders API

폴더는 메모를 주제별로 정리하기 위한 기능이다.

### 8.1 폴더 생성

`POST /api/folders`

#### Request Body

| 필드 | 타입 | 필수 | 설명 |
|---|---|---|---|
| name | String | O | 폴더 이름 |
| parentId | Long | 선택 | 상위 폴더 id |

#### Request 예시

| 필드 | 값 |
|---|---|
| name | 공부 |
| parentId | null |

#### Response

| 필드 | 타입 | 설명 |
|---|---|---|
| id | Long | 폴더 id |
| name | String | 폴더 이름 |
| parentId | Long | 상위 폴더 id |
| createdAt | LocalDateTime | 생성일시 |
| updatedAt | LocalDateTime | 수정일시 |

#### Validation

| 필드 | 조건 |
|---|---|
| name | 필수, 1자 이상 100자 이하 |
| parentId | 선택 |

---

### 8.2 폴더 목록 조회

`GET /api/folders`

#### Response

| 필드 | 타입 | 설명 |
|---|---|---|
| id | Long | 폴더 id |
| name | String | 폴더 이름 |
| parentId | Long | 상위 폴더 id |
| createdAt | LocalDateTime | 생성일시 |
| updatedAt | LocalDateTime | 수정일시 |

---

### 8.3 폴더 이름 수정

`PATCH /api/folders/{folderId}`

#### Request Body

| 필드 | 타입 | 필수 | 설명 |
|---|---|---|---|
| name | String | O | 수정할 폴더 이름 |

#### Response

| 필드 | 타입 | 설명 |
|---|---|---|
| id | Long | 폴더 id |
| name | String | 수정된 폴더 이름 |
| parentId | Long | 상위 폴더 id |
| createdAt | LocalDateTime | 생성일시 |
| updatedAt | LocalDateTime | 수정일시 |

---

### 8.4 폴더 삭제

`DELETE /api/folders/{folderId}`

#### Response

`204 No Content`

#### 설명

폴더 삭제 정책은 추후 결정한다.

가능한 정책은 다음과 같다.

1. 폴더 안에 메모가 있으면 삭제 불가
2. 폴더 삭제 시 메모는 폴더 없음 상태로 변경
3. 폴더 삭제 시 하위 메모까지 함께 삭제

초기 MVP에서는 2번 정책을 권장한다.

---

## 9. Tags API

태그는 메모를 유연하게 분류하기 위한 기능이다.

초기에는 사용자가 직접 태그를 생성하는 API보다, 메모 저장 시 태그를 함께 저장하는 방식을 고려한다.

### 9.1 태그 목록 조회

`GET /api/tags`

#### Response

| 필드 | 타입 | 설명 |
|---|---|---|
| id | Long | 태그 id |
| name | String | 태그 이름 |
| noteCount | Long | 해당 태그가 연결된 메모 수 |

---

### 9.2 특정 태그의 메모 조회

`GET /api/tags/{tagName}/notes`

#### Response

| 필드 | 타입 | 설명 |
|---|---|---|
| id | Long | 메모 id |
| title | String | 메모 제목 |
| contentPreview | String | 본문 일부 미리보기 |
| favorite | Boolean | 즐겨찾기 여부 |
| createdAt | LocalDateTime | 생성일시 |
| updatedAt | LocalDateTime | 수정일시 |

---

### 9.3 메모 태그 수정

`PUT /api/notes/{noteId}/tags`

#### Request Body

| 필드 | 타입 | 필수 | 설명 |
|---|---|---|---|
| tags | List<String> | O | 메모에 연결할 태그 목록 |

#### Request 예시

| 필드 | 값 |
|---|---|
| tags | study, memo, spring |

#### Response

| 필드 | 타입 | 설명 |
|---|---|---|
| noteId | Long | 메모 id |
| tags | List<String> | 연결된 태그 목록 |

#### 설명

- 기존 태그 연결을 요청값 기준으로 교체한다.
- 존재하지 않는 태그는 새로 생성한다.
- 인증 적용 후에는 로그인한 사용자의 태그만 관리한다.

---

## 10. Search API

메모 검색 기능을 제공한다.

초기에는 제목과 본문을 기준으로 검색한다.

추후 태그, 폴더, 즐겨찾기 필터를 추가할 수 있다.

### 10.1 메모 검색

`GET /api/search?query={keyword}`

#### Request 예시

`GET /api/search?query=Spring`

#### Query Parameters

| 이름 | 필수 | 설명 |
|---|---|---|
| query | O | 검색어 |
| folderId | 선택 | 특정 폴더 안에서 검색 |
| tag | 선택 | 특정 태그 기준 검색 |
| favorite | 선택 | 즐겨찾기 여부 |
| page | 선택 | 페이지 번호 |
| size | 선택 | 페이지 크기 |

#### Response

| 필드 | 타입 | 설명 |
|---|---|---|
| id | Long | 메모 id |
| title | String | 메모 제목 |
| contentPreview | String | 본문 일부 미리보기 |
| favorite | Boolean | 즐겨찾기 여부 |
| folderId | Long | 폴더 id |
| tags | List<String> | 태그 목록 |
| createdAt | LocalDateTime | 생성일시 |
| updatedAt | LocalDateTime | 수정일시 |

#### Error

| 상황 | HTTP Status | Code |
|---|---|---|
| 검색어가 비어 있음 | 400 | INVALID_REQUEST |

---

## 11. 초기 구현 우선순위

현재 개발 단계에서는 모든 API를 한 번에 구현하지 않는다.

### Phase 1. 기본 서버

- `GET /api/health`

### Phase 2. 인증 없는 Note CRUD

- `POST /api/notes`
- `GET /api/notes`
- `GET /api/notes/{noteId}`
- `PATCH /api/notes/{noteId}`
- `DELETE /api/notes/{noteId}`

### Phase 3. 공통 예외 처리

- `NOTE_NOT_FOUND`
- `INVALID_REQUEST`
- `GlobalExceptionHandler`

### Phase 4. DB 연결

- JPA 설정
- PostgreSQL 설정
- NoteRepository 적용

### Phase 5. 사용자 인증

- `POST /api/auth/signup`
- `POST /api/auth/login`
- `GET /api/me`

### Phase 6. 정리 기능

- Folder API
- Tag API

### Phase 7. 검색 기능

- `GET /api/search`

---

## 12. 현재 구현 상태

| API | 상태 |
|---|---|
| `GET /api/health` | 완료 |
| `POST /api/notes` | 예정 |
| `GET /api/notes` | 예정 |
| `GET /api/notes/{noteId}` | 예정 |
| `PATCH /api/notes/{noteId}` | 예정 |
| `DELETE /api/notes/{noteId}` | 예정 |
| Auth API | 예정 |
| Folder API | 예정 |
| Tag API | 예정 |
| Search API | 예정 |

---

## 13. API 설계 원칙

- API 경로는 명확하고 예측 가능하게 작성한다.
- 명사는 복수형을 사용한다.
- 생성은 `POST`, 조회는 `GET`, 수정은 `PATCH`, 삭제는 `DELETE`를 사용한다.
- 인증 도입 후 모든 사용자 데이터는 로그인한 사용자 기준으로 제한한다.
- 초기 구현은 단순하게 만들고, 이후 공통 응답과 예외 처리를 도입한다.
- 한 번에 모든 API를 구현하지 않고, Note CRUD부터 구현한다.
