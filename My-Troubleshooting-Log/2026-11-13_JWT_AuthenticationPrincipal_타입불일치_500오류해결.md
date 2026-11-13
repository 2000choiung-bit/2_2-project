# 📝 [Troubleshooting] 추천 API 호출 시 JWT @AuthenticationPrincipal 타입 불일치 500 오류 해결

## 1. 개요 (Overview)
* **작업 내용:** AI 사이즈 추천 API(`GET /api/recommendations/{productId}`)를 JWT 토큰 인증 기반으로 전환하는 과정에서 발생한 `500 Internal Server Error` 해결
* **관련 기술:** Spring Boot, Spring Security, JWT, Swagger UI

## 2. 발생한 문제 및 해결 과정 (Trouble Shooting)

### 🔴 문제: 로그인 토큰을 포함해 추천 API 호출 시 500 에러(`NullPointerException`) 발생
* **원인:** 보안 강화를 위해 기존 `@RequestParam Long userId` 방식을 제거하고 `@AuthenticationPrincipal UserDetails userDetails`로 변경했으나, 프로젝트의 `JwtAuthenticationFilter`는 `SecurityContext`에 `UserDetails` 객체가 아닌 `String username`을 Principal로 저장하고 있었음. 이로 인해 `userDetails`가 `null`로 주입되면서 `userDetails.getUsername()` 호출 시 NPE가 발생함.
* **해결:** `SizeRecommendationController`의 파라미터 타입을 `@AuthenticationPrincipal String username`으로 일치시켜 토큰의 사용자 식별자를 정상적으로 서비스 계층에 전달하도록 수정함.
* **느낀점:** 공통 보안 필터(`JwtAuthenticationFilter`)에서 `Authentication` 객체에 적재하는 Principal의 실제 타입을 먼저 확인하고 컨트롤러 바인딩 타입을 맞춰야 불필요한 런타임 에러를 막을 수 있다.
