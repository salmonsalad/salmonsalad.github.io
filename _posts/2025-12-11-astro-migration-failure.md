---
title: Astro Paper 마이그레이션 실패기
date: 2025-12-11 18:30:00 +0900
categories: [Dev, Troubleshooting]
tags: [astro, jekyll, migration, github-actions, failure-story]
---

Jekyll에서 **Astro Paper** 테마로 마이그레이션을 시도했으나, 여러 가지 기술적 문제로 인해 다시 **Jekyll**로 복귀했다. 그 과정에서 발생한 문제점들과 해결 시도 과정을 기록으로 남긴다.

---

## 1. 저장소 브랜치(Branch) 혼선

**📉 상황**  
저장소를 확인해보니 `master`, `main`, `gh-pages` 등 여러 브랜치가 혼재되어 있었다. Jekyll은 주로 레거시 브랜치인 `master`를 사용하고, Astro는 `main`을 기본으로 사용하다 보니 기준 브랜치를 정하기가 모호했다.

**🧐 원인**  
기존 Jekyll 프로젝트 위에 Astro를 덮어씌우는 방식으로 진행하려다 보니, 구형/신형 브랜치 전략이 충돌했다.

**💡 결과**  
브랜치 정리가 선행되어야 함을 확인했다.

<br>

## 2. GitHub Pages 빌드 소스 설정 미스

**📉 상황**  
Astro 파일을 `main` 브랜치에 푸시했지만, 라이브 사이트에는 변경 사항이 반영되지 않았다.

**🧐 원인**  
GitHub Pages 설정의 **Build Source**가 여전히 `master` 브랜치로 고정되어 있었다. `main`에 코드를 올려도 배포 프로세스는 `master`를 바라보고 있었기 때문이다.

**💡 결과**  
Repository Settings에서 빌드 소스를 변경해야 함을 인지했다.

<br>

## 3. 파일 구조 및 설정 파일 충돌

**📉 상황**  
`_config.yml` (Jekyll)과 `astro.config.mjs` (Astro) 파일이 한 루트 디렉토리에 섞여 있어 프로젝트 구조가 매우 지저분해졌다.

**🧐 원인**  
기존 환경을 깨끗하게 정리(Clean)하지 않고 마이그레이션을 진행하여, 레거시 파일과 신규 파일이 뒤섞이는 문제가 발생했다.

**💡 결과**  
완전한 디렉토리 분리 혹은 기존 파일 삭제 후 진행이 필요했다.

<br>

## 4. GitHub Actions (CI/CD) 배포 실패

가장 결정적인 문제는 자동 배포 단계에서 발생했다.

### 💥 실패 1: 빌드 환경 불일치
*   **현상**: 커밋 푸시 후 Actions 위젯에 빨간색 실패(Failure) 에러가 발생했다.
*   **원인**: 코드는 Astro 기반인데, GitHub Actions의 워크플로우는 여전히 Jekyll 빌드 환경(Ruby/Jekyll v3.8)으로 설정되어 있었다.
*   **조치**: `deploy.yml`을 생성하여 Astro용 Node.js 빌드 환경으로 교체했다.

### 💥 실패 2: 패키지 매니저 충돌 (npm vs pnpm)
*   **현상**: 환경을 교체했으나 `Error: No pnpm version is specified` 에러가 발생했다.
*   **원인**: 로컬 개발 환경은 `npm`을 사용했지만, Astro 템플릿에 포함된 `pnpm-lock.yaml` 파일이 저장소에 함께 올라갔다. CI 서버가 이를 감지하고 pnpm으로 빌드를 시도했으나 버전 정보가 없어 실패했다.
*   **조치**: 락파일(Lockfile)을 하나로 통일해야 했으나, 이미 꼬인 설정으로 인해 피로도가 누적되었다.

---

## 🏁 결론

결국 Astro로의 전환은 **환경 설정의 복잡성**과 **기존 데이터와의 충돌**로 인해 중단했다. 대신 관리가 용이하고 완성도가 높은 **Jekyll Chirpy** 테마를 도입하기로 결정했다. 

새로운 기술 도입도 좋지만, 때로는 **안정적이고 익숙한 도구**가 운영 측면에서 더 효율적일 수 있음을 확인했다.
