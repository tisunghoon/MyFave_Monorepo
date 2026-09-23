# MyFave — 인플루언서 플리마켓 커머스 (모노레포)

`backend/` (Spring Boot 3.4.1 / Java 21) · `frontend/` (React 19 + TS + Vite) · `observability/` (Grafana 스택) · `backend/load-test/` (k6).
하위 폴더 작업 시 해당 폴더의 AGENTS.md를 먼저 읽어라. 규칙 충돌 시 하위 파일이 우선.

## 인프라 기동 (실측: docker-compose.yml)
- 앱 의존 인프라(PostgreSQL 16 + Redis 7): `docker-compose up -d`
- 모니터링(Prometheus/Grafana/Loki/Tempo/Promtail): `docker-compose -f docker-compose.monitoring.yml up -d`
- DB는 PostgreSQL 단일. MySQL은 쓰지 않는다.

## Git 전략 · PR 규약
- Issue 먼저 생성(`.github/ISSUE_TEMPLATE/` 사용) → 관련 브랜치에서 작업 → PR.
- PR은 `.github/PULL_REQUEST_TEMPLATE.md`를 채우고 "무엇을/왜(Why)"를 반드시 적는다. 머지는 최소 1인 승인.
- push 전 `git pull`로 충돌을 로컬에서 먼저 해소한다.

## Contract-First 공통 계약 (FE·BE 공유)
- 모든 API 응답은 `global/common/ApiResponse` 봉투: `{ code:int, message, errorCode, data }`. 성공은 `errorCode=null`.
- 에러는 `global/error/ErrorCode` enum. 프론트는 `errorCode`로 분기(예: `PRODUCT_SOLD_OUT`).
- 401(인증 만료)=토큰 재발급 시도 / 403(권한·한도)=안내만. 빈 목록은 `[]`, 없는 객체는 `null`.

## 검증 (완료 판정 — 스스로 돌려서 확인)
- 백엔드: `cd backend && ./gradlew build` (테스트 포함) 통과.
- 프론트: `cd frontend && npm run lint && npm run build` 통과.
- 변경 완료를 주장하기 전에 위 명령이 실제로 통과했는지 출력으로 확인한다.

## 경계
- always: 커밋 전 해당 파트 빌드·테스트·린트 통과 확인 / API 변경 시 FE·BE 계약 동기화.
- ask first: DB 스키마 변경, `application-chaos.yml` 튜닝값 변경, 새 외부 의존성 추가.
- never: `.env`·키·토큰·`postgresql.example` 채운 값 커밋 / 서킷·카오스 설정 임의 변경 / main 직접 push.

## 관련 위치 (긴 세부는 여기 참조)
- 부하/카오스 드릴: `backend/load-test/README.md` · 관측 스택: `observability/`
