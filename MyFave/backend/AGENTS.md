# MyFave Backend — Spring Boot 3.4.1 / Java 21

## 명령어 (실측: build.gradle, gradlew)
- 빌드(테스트 포함): `./gradlew build`  ·  빌드만: `./gradlew build -x test`  ·  클린: `./gradlew clean`
- 실행: `./gradlew bootRun` (기본 프로파일 local)
- 단일 테스트: `./gradlew test --tests "*.OrderServiceTest"`
- 부하테스트: `SPRING_PROFILES_ACTIVE=loadtest ./gradlew bootRun` (1000유저 자동 시드)

## 아키텍처
- 패키지: `com.myfave.api` 아래 `domain/{auth,user,product,cart,order,payment,coupon,shipping,chat,saleevent,content}` + `global/{config,security,common,error,filter,chaos,...}`.
- 레이어드(Controller-Service-Repository), 도메인별 패키지. 컨트롤러 비즈니스 로직 금지, DI는 생성자 주입.
- 모든 API는 `/api/v1` 하위. 응답은 `global/common/ApiResponse`, 에러는 `global/error/ErrorCode` enum.

## 이 프로젝트만의 규칙 (코드만 봐선 모름)
- IMPORTANT: 시간 타입은 DB `TIMESTAMPTZ` + Java `OffsetDateTime`. `LocalDateTime` 금지.
- `CartItem`·`ChatRoom`·`OrderItem`·`ShortForm`·`StyleFeed`는 `BaseEntity` 미상속 → `updated_at` 없음. UPDATE 시 수동 갱신.
- 결제=Saga: 실패 시 보상 트랜잭션(주문 상태 복구) 필수. `final_payment_id`는 결제 성공 시점에만 UPDATE.

## DB 스키마 (마이그레이션 툴 없음 — 함정)
- Flyway/Liquibase 없음. 스키마는 `scripts/db/*.sql`를 손으로 실행, JPA는 검증만(`application.yml` ddl-auto=validate).
- local 프로파일만 ddl-auto=update. 운영/드릴에서 엔티티로 스키마 바꾸지 말고 `scripts/db`에 SQL 추가.

## 카오스 드릴 (chaos 프로파일 — 평소엔 없음)
- 실행: `SPRING_PROFILES_ACTIVE=local,chaos ./gradlew bootRun`
- 장애 주입 빈·`ChaosController`는 `@Profile("chaos")` 전용. 런타임 토글: `POST /api/v1/internal/chaos/pg`.
- 서킷/레이트리미터 튜닝값은 `application-chaos.yml`에 있고 SLO 기준 — 변경은 ask first.

## 검증 (완료 판정)
- 변경 후 `./gradlew build` 통과를 출력으로 확인한 뒤에만 완료로 본다.
