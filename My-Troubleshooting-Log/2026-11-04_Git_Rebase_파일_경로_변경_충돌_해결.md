# 🪵 [Troubleshooting] Git Rebase 경로 변경 충돌 및 Feature 브랜치 관리 전략

## 1. 개요 (Overview)
* **작업 내용**: Git Rebase 중 디렉토리 구조 변경으로 인한 삭제/충돌 파일 해소 및 피처 브랜치 전략 수립
* **관련 기술**: Git/GitHub, IntelliJ IDEA Git Tool, Spring Boot 3.x

## 2. 발생한 문제 및 해결 과정 (Trouble Shooting)

### 🔴 문제 1: 팀원 폴더 구조 변경으로 인한 Git Rebase 파일 삭제 및 경로 충돌
* **원인**: 팀원의 프로젝트 루트 경로 이동(`backend/src/...`) 커밋과 내 기존 경로 커밋이 부딪히며 `AuthController.java` 삭제 인식 및 `SecurityConfig.java`, `UserController.java` 충돌 발생.
* **해결**: 
  * IntelliJ 충돌 해결 UI를 통해 신규 보안 코드 및 컨트롤러는 `[내 변경 내용 수락(Accept Yours)]` 선택.
  * 구 버전 파일(`AuthController.java`)은 `[다른 이의 변경 내용 수락(Accept Theirs)]`을 통해 안전하게 삭제 처리 수용.
  * `git add .` ➡️ `git rebase --continue` ➡️ `git push --force-with-lease` 순서로 이력 깔끔히 정돈 후 푸시 완료.
* **느낀점**: 구조 변경 커밋과 병합할 때 삭제된 파일과 수정한 파일의 처리 방향을 명확히 잡는 것이 코드 손실을 막는 핵심임을 깨달음.

### 🔴 문제 2: 후속 작업 진행 시 PR 코드 엉킴 방지를 위한 브랜치 분리
* **원인**: 이미 PR을 올린 브랜치에서 다음 작업(JWT)을 계속 진행할 경우, 리뷰 중인 PR에 추가 커밋이 섞여 리뷰 파이프라인이 오염되는 문제 발생 가능.
* **해결**: PR 생성 직후 `git checkout -b feature/jwt-auth`로 별도 작업 공간(Branch)을 분리하여 기능 단위 PR의 독립성을 확보함.
