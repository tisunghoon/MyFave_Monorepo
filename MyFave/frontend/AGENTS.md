# MyFave Frontend — React 19 + TypeScript + Vite

## 명령어 (실측: package.json)
- 개발: `npm run dev`  ·  빌드+타입체크: `npm run build` (`tsc -b && vite build`)
- 린트: `npm run lint` (max-warnings 0)  ·  자동수정: `npm run lint:fix`  ·  포맷: `npm run format`
- 패키지 매니저 npm(package-lock.json). 커밋 전 lint 에러 0, build 성공 확인.

## 상태 관리 경계 (이 프로젝트 규칙)
- 서버 상태 → TanStack Query (`useState+useEffect` fetch 금지) · 전역 클라이언트 상태 → Zustand · URL 상태 → Router search params. Redux 미사용.
- 폼은 `react-hook-form` + `zod` 고정. 라우터는 `createBrowserRouter`.

## API 계약 (백엔드 공통 봉투)
- 응답은 `{ code, message, errorCode, data }`. API 함수는 `data.data`까지 벗겨 반환.
- 에러 분기는 `errorCode` 기준(예: `PRODUCT_SOLD_OUT`). 401=토큰 재발급 / 403=안내 모달.
- 환경변수는 `import.meta.env.VITE_API_BASE_URL`만 사용. URL 하드코딩 금지.

## 컨벤션
- 컴포넌트 `PascalCase.tsx`, 훅/유틸 `camelCase.ts`, 폴더 `kebab-case`. import 순서·`@/` alias는 ESLint가 강제.
- 모바일 퍼스트(가로 최대 480px). Tailwind + `clsx`.

## 경계 & 함정
- never: `console.log` 커밋 / localStorage에 민감정보 저장 / URL 하드코딩.
- `<form>` 내부 `<button>`은 type 명시(미지정 시 기본 submit).

## 검증 (완료 판정)
- 변경 후 `npm run lint && npm run build` 통과를 확인한 뒤에만 완료로 본다.
