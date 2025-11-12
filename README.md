# Wemmy-Backend

서울시 임신, 육아 맞춤 혜택 제공 플랫폼 'Wemmy'

사용자의 거주 지역, 가족 구성, 임신/양육 상태를 기반으로 정부 및 지방자치단체의 복지 혜택 정보 및 복지시설 정보를 제공
## 프로젝트 개요

### 수상 및 기본 정보
- **수상**: 동양미래대학교 스마트 프로젝트 경진대회 우수상
- **개발 기간**: 2024.03 ~ 2024.11 (9개월)
- **팀 구성**: 6명 (백엔드 1명, WEB 1명, APP 2명, 크롤링 1명, 기획 1명)
- **개인 기여도**: 80% (백엔드 단독 개발)

### 기술 스택
- Backend : Spring Boot 3.2.5, Java 17
- Security : Spring Security 6.x, JWT, OAuth2.0
- Database : MySQL, JPA/Hibernate, JPQL
- API Documentation : Swagger (SpringDoc OpenAPI 3.0)
- External API : OpenFeign, Kakao OAuth2.0, Government Open API
- Testing : JUnit 5, Mockito, AssertJ
- Others : Jasypt (암호화), Lombok

### 프로젝트 규모
- **DB 테이블**: 16개
- **전체 코드**: 8,072줄
- **API 엔드포인트**: 30개 이상

## 개발 배경

### 해결하고자 한 문제

**1. 복지 정보 접근성 부족**
- 전국의 복지 혜택 정보가 중앙정부, 지방자치단체에 산재되어 있어 통합 조회가 어려움
- 임산부 및 양육자가 본인에게 적합한 혜택을 찾기 위해 여러 사이트를 방문해야 하는 불편함

**2. 정보 탐색 시간 부족**
- 육아 중인 부모는 시간적 여유가 없어 복지 정보를 충분히 탐색하기 어려움
- 복잡한 신청 자격 요건으로 인해 본인이 대상인지 판단하기 어려움

**3. 지역별 혜택 차이**
- 거주 지역에 따라 제공되는 복지 혜택이 상이하나, 이를 한눈에 비교하기 어려움
- 주변 육아 편의시설(수유실, 놀이터 등) 정보를 실시간으로 찾기 어려움

### 해결 방안

**1. 통합 복지 정보 플랫폼**
- 정부 Open API 연동을 통해 전국 복지 혜택 정보를 하나의 플랫폼에 통합
- 4단계 행정구역 계층(권역-시도-시군구-읍면동)을 통한 정확한 지역별 혜택 제공

**2. 사용자 맞춤형 필터링**
- 거주 지역, 임신/육아 여부, 가족 구성, 소득 수준을 기반으로 적합한 혜택만 자동 필터링
- 사용자가 직접 자격 요건을 확인할 필요 없이 맞춤 추천 제공

**3. GPS 기반 실시간 시설 검색**
- Haversine Formula를 활용한 반경 기반 육아 시설 검색
- 외출 중에도 주변 수유실, 놀이터 등을 빠르게 찾을 수 있는 편의성 제공

## 개인 역할 및 기여

### 담당 업무 (백엔드 단독 개발)

**1. 백엔드 아키텍처 설계 및 구현**
- Spring Boot 3.2.5 기반 RESTful API 서버 구축
- Layered Architecture 적용 및 JPA/Hibernate 기반 데이터 연동

**2. 인증/인가 시스템 구현**
- Spring Security + JWT(Access/Refresh Token) 기반 인증
- Kakao OAuth2.0 소셜 로그인 연동

**3. 주요 기능 API 개발**
- 복지 혜택: 사용자 맞춤형 추천(지역/가족/소득 기반), 신청 및 중복 검증
- 시설 정보: 4단계 지역 계층 구조, 좌표 기반 반경 검색(카테고리별 그룹화)
- 공통: 조회수 기반 인기도 관리 및 스크랩 기능

**4. 외부 데이터 연동 및 자동화**
- OpenFeign 활용 정부 Open API 연동 (지역정보)
- CommandLineRunner 기반 초기 데이터 자동 로딩 및 관리자 계정 생성

**5. 개발 환경 및 품질 관리**
- Swagger(SpringDoc OpenAPI 3.0) 기반 API 명세서 작성
- 전역 예외 처리 및 커스텀 ErrorCode 관리를 통한 예외 응답 처리 통일
- Mockito/JUnit 5 기반 단위 테스트 작성

## 기술적 도전 및 해결

### 1. Spring Security 마이그레이션

**문제**
- 기존 Interceptor 기반 인증 방식은 Spring Security 생태계와 통합이 어려워 확장성 제한
- OAuth2.0 소셜 로그인 도입 시 표준 지원 불가

**해결**
- Spring Security Filter Chain 기반 인증 시스템으로 전면 마이그레이션
- `JwtAuthenticationFilter` 구현으로 JWT 토큰 검증 자동화
- `@PreAuthorize` 어노테이션을 활용한 역할 기반 접근 제어 적용

### 2. JPA N+1 문제로 인한 성능 저하

**문제**
- 복지 혜택 조회 시 연관 엔티티(지역, 카테고리) 조회로 인해 25개 쿼리 실행
- API 응답 속도 평균 1.2초로 사용자 경험 저하

**해결**
- JPQL + Fetch Join을 활용한 연관 엔티티 즉시 로딩
- 쿼리 실행 횟수 25개 → 5개로 감소 (80% 개선)

### 3. GPS 기반 반경 검색 성능 최적화

**문제**
- 전국 5,000개 육아 시설 데이터에서 반경 검색 시 응답 속도 2.5초
- 모든 시설에 대해 거리 계산을 수행하여 비효율적

**해결**
- Haversine Formula를 활용한 거리 계산 알고리즘 구현
- 위도/경도 컬럼에 Spatial Index 적용
- 검색 응답 속도 2.5s → 1.0s (60% 개선)

## 아키텍처

### Layered Architecture
**Controller → Service → Repository → Domain** 구조로 계층별 책임 분리

### 주요 패키지 구조
- **api**: REST API Controllers (user, benefit, scrap, facility, admin 등 7개 도메인)
- **domain**: JPA Entities (16개 테이블에 대응하는 엔티티)
- **dto**: Data Transfer Objects (요청/응답 DTO)
- **repository**: JPA Repositories (데이터 접근 계층)
- **service**: Business Logic (비즈니스 로직 처리)
- **global**: 전역 설정 (Security, JWT, OAuth, 예외 처리, Jasypt 암호화)

### 보안 아키텍처
- **Spring Security Filter Chain**: `JwtAuthenticationFilter` → `UsernamePasswordAuthenticationFilter`
- **JWT 기반 인증**: Access Token, Refresh Token
- **OAuth2.0**: Kakao 소셜 로그인 Factory 패턴 구현
- **역할 기반 접근 제어**: USER, ADMIN 권한 분리

## 핵심 기능

### 1. 회원 관리
- **회원가입/로그인**: 이메일 기반 회원가입, Kakao OAuth2.0 소셜 로그인
- **온보딩 정보 관리**: 거주 지역(시군구), 가족 구성, 임신/육아 상태, 가구원 수, 월소득 등록
- **JWT 인증**: Access Token, Refresh Token 발급 및 갱신

### 2. 맞춤형 복지 혜택 추천
- **자동 필터링**: 거주 지역, 임신/육아 여부, 가족 구성, 소득 기반 맞춤 추천
- **카테고리 분류**: 임산부(카테고리 1), 영유아(카테고리 2) 자동 분류
- **혜택 제공**: 자치구 복지 혜택, 정부 복지 혜택, 지역 프로그램 정보 통합 제공
- **스크랩 기능**: 관심 복지 혜택 저장 및 관리, 중복 신청 검증

### 3. GPS 기반 육아 시설 검색
- **반경 검색**: Haversine Formula 활용 좌표 기반 거리 계산
- **카테고리별 그룹화**: 수유실, 어린이집 등 시설 유형별 분류 및 반환
- **실시간 조회**: 위도, 경도, 반경 입력으로 주변 육아 편의시설 즉시 검색

### 4. 지역 기반 서비스
- **3단계 행정구역 계층**: 시도 → 시군구 → 읍면동 구조
- **지역별 혜택 필터링**: 사용자 거주지 기반 자동 필터링
- **관리자 데이터 수집**: 정부 Open API 연동 지역 정보 수집 및 저장

## 인증 및 보안
### Spring Security
- Filter Chain 기반 인증 아키텍처
- JWT 인증 필터: `JwtAuthenticationFilter`
- 역할 기반 접근 제어 (RBAC): USER, ADMIN

### JWT 토큰
- Stateless 세션 관리
- Claims: id, email, userRole
- Bearer 토큰 방식

### Jasypt 암호화
- 설정 파일의 민감 정보 암호화

### OAuth2.0
- Kakao 소셜 로그인 연동
- Factory 패턴으로 다중 OAuth 제공자 확장 가능

## 데이터베이스 구조

### 주요 테이블
- **USERV2**: 사용자 정보 (이메일, 비밀번호, 거주 지역, 온보딩 데이터)
- **welfare**: 복지 혜택 정보 (제목, 내용, 신청 방법, 지역, 카테고리)
- **program**: 지역별 프로그램 정보 (교육 프로그램 등)
- **scrap**: 사용자 스크랩 정보
- **facility**: 육아 시설 정보 (위치, 주소, 카테고리)
- **baby**: 아기 정보
- **welfare_registration**: 복지 혜택 신청 내역
- **program_registration**: 프로그램 신청 내역
- **regions**: 권역 정보
- **sido_areas**: 시도 정보
- **sigu_areas**: 시군구 정보
- **umd_areas**: 읍면동 정보
- **wcategory**: 복지 카테고리 (임산부, 영유아 등)

# 기대 효과

### 사회적 영향
- **서울시 25개 자치구 복지 혜택 정보 통합**으로 시민들의 복지 접근성 향상
- **정부 복지 혜택 실시간 연동**을 통한 최신 정보 제공으로 복지 사각지대 해소
- **서울시 육아 편의시설 정보 통합**으로 육아 가정의 편의성 증대
- **3단계 행정구역 계층 구조**를 통한 지역 맞춤형 서비스로 복지 혜택 활용도 제고

### 기술적 성과
- **백엔드 단독 개발**로 전체 시스템 아키텍처 설계 및 구현 완료
- **30개 이상 REST API 설계 및 문서화**로 확장 가능한 서비스 기반 구축
- **Swagger API 문서 자동화**를 통한 개발팀 간 협업 효율성 극대화
- **테스트 커버리지 73% 달성**으로 안정적이고 유지보수 가능한 서비스 기반 마련

### 성능 및 품질 개선
- **API 응답 속도 40% 개선** (1.2s → 0.7s)으로 사용자 경험 향상
- **JPA N+1 문제 해결**로 데이터베이스 쿼리 80% 감소 (25개 → 5개)
- **GPS 기반 시설 검색 응답 속도 60% 개선** (2.5s → 1.0s)으로 실시간 위치 기반 서비스 최적화
- **주요 비즈니스 로직 단위 테스트 자동화**로 서비스 안정성 확보

### 보안 및 데이터 관리
- **Jasypt 암호화 및 Spring Security 표준 아키텍처 적용**으로 민감 정보 보호
- **JWT 기반 Stateless 인증**으로 확장 가능한 세션 관리 체계 구축
- **크롤링 데이터 파싱 및 Open API 연동**으로 실시간 데이터 동기화 자동화

### 협업 및 프로젝트 관리
- **6명 팀 프로젝트에서 백엔드 80% 기여**로 핵심 개발 주도
- **API 명세 기반 병렬 개발**로 9개월 프로젝트 기간 단축 및 효율적 협업 달성

## 프로젝트를 통해 배운 점

### 1. 실전 Spring Boot 개발 역량 향상

**Spring Security 깊이 있는 이해**
- Interceptor 기반 인증에서 Filter Chain 기반으로 마이그레이션하며 Spring Security의 동작 원리 이해
- JWT 토큰 기반 Stateless 인증 시스템 설계 및 구현 경험
- 역할 기반 접근 제어(RBAC)를 통한 권한 관리 체계 구축

**JPA 성능 최적화 실전 경험**
- N+1 문제를 직접 경험하고 JPQL, Fetch Join을 활용한 해결 방법 체득
- 쿼리 실행 횟수와 응답 속도 간의 상관관계 이해
- Spatial Index를 활용한 위치 기반 검색 최적화

### 2. 단독 백엔드 개발 경험

**전체 개발 생명주기 주도**
- 요구사항 분석부터 설계, 구현, 테스트, 배포까지 전체 과정을 주도적으로 진행
- 16개 테이블, 30개 이상의 API 엔드포인트 설계 및 구현
- 아키텍처 설계 단계에서의 의사결정 경험 (Layered Architecture, 보안 전략 등)

**문제 해결 능력 향상**
- 성능 문제(N+1, 반경 검색 속도)를 직접 발견하고 해결하는 과정에서 디버깅 능력 향상
- 기술 선택의 기준(Interceptor vs Filter Chain)을 고민하고 적용하는 경험

### 3. 외부 시스템 연동 경험

**OpenFeign을 활용한 선언적 API 통신**
- 정부 Open API 연동을 통해 외부 시스템과의 통신 패턴 학습
- Connection Pool, Timeout 설정 등 HTTP 클라이언트 설정 경험
- 에러 핸들링 및 재시도 전략 구현

**OAuth2.0 소셜 로그인 구현**
- Kakao OAuth2.0 인증 플로우 이해 및 구현
- Access Token 교환, 사용자 정보 조회 등 표준 OAuth 프로세스 경험

### 4. 협업 및 커뮤니케이션 능력

**API 문서화의 중요성**
- Swagger를 통한 API 문서 자동화로 프론트엔드 팀과의 협업 효율성 향상
- API 명세를 명확히 하는 것이 전체 개발 속도에 미치는 영향 체감

### 5. 테스트의 중요성 인식

**단위 테스트 작성 습관화**
- JUnit 5, Mockito를 활용한 Service Layer 테스트 작성
- 테스트 커버리지 73% 달성을 통해 코드 품질 향상
- 리팩토링 시 기존 기능 보장을 위한 테스트의 가치 체감

## 향후 개선 계획

### 1. 계층 간 책임 분리 강화
- **현재 상태**: Controller Layer에서 DTO 생성 로직이 일부 포함
- **개선 목표**: 객체 생성 책임을 Service Layer로 위임하여 단일 책임 원칙(SRP) 준수
- **학습 포인트**: Layered Architecture에서 각 계층의 명확한 역할 분리 중요성 인식

### 2. Static Factory Method 패턴 적용
- **현재 상태**: Service Layer에서 반복적인 Builder 패턴 사용
- **개선 목표**: DTO에 Static Factory Method 도입으로 객체 생성 로직 캡슐화
- **기대 효과**: 코드 중복 70% 감소, 테스트 용이성 향상

### 3. 성능 최적화 및 코드 개선
- **이미지 URL 매핑**: if-else 체인(53줄) → Map 기반 설정으로 개선
- **스크랩 체크 로직**: O(n²) → O(n) 알고리즘 최적화 (2중 for문 -> stream으로 필터링)
- **중복 메서드 제거**: Entity → DTO 변환 로직을 유틸 메서드로 통합
