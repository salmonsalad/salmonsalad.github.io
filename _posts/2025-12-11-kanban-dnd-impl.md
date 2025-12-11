---
title: "[칸반 프로젝트] 1주차 Day 2: 드래그 앤 드롭(Dnd) 구현"
date: 2025-12-11 10:00:00 +0900
categories: [Project, Kanban Board]
tags: [react, dnd-kit, typescript, troubleshooting]
---

## 1. 개요
본 프로젝트는 **"Interactive Kanban Board"** 개인 포트폴리오 작업의 일환으로 진행되었다.
단순한 투두 리스트를 넘어, **드래그 앤 드롭(Drag & Drop)** 기반의 직관적인 UX를 제공하고, **전역 상태 관리**를 통해 데이터를 효율적으로 다루는 것을 목표로 한다.

Day 2에서는 프로젝트 초기 환경 설정부터 핵심 기능인 칸반 보드의 CRUD 및 dnd 로직 구현까지 진행했다.

---

## 2. 기술 스택 선정 및 의사결정

### 2.1 Framework & Language
- **Vite + React (TypeScript)**
    - CRA(Create React App) 대비 빠른 빌드 속도와 HMR(Hot Module Replacement)을 위해 Vite를 선택했다.
    - 정적 타입 검사를 통해 런타임 에러를 사전에 방지하고 유지보수성을 높이기 위해 TypeScript를 도입했다.

### 2.2 State Management: Zustand
- **선정 이유**: Redux는 보일러플레이트 코드가 많고 설정이 복잡한 반면, Zustand는 Hook 기반의 간결한 API를 제공하여 React 컴포넌트와의 결합도가 낮다. 또한 번들 사이즈가 매우 작아 가벼운 칸반 보드 프로젝트에 적합하다고 판단했다.
- **활용**: `useBoardStore`를 생성하여 컬럼(Column)과 태스크(Task)의 상태를 중앙에서 관리하고, `persist` 미들웨어를 활용해 `localStorage`에 데이터를 영구 저장하도록 구현했다.

### 2.3 Drag & Drop: @dnd-kit
- **선정 이유**: 기존에 많이 사용되던 `react-beautiful-dnd`는 유지보수가 중단된 상태다. 이에 비해 `@dnd-kit`은 모듈화가 잘 되어 있고, Headless UI 방식으로 스타일링의 자유도가 높아 최신 React 생태계 표준으로 자리 잡고 있어 선택했다.
- **구현 과제**: `DndContext`, `SortableContext` 등을 활용해 드래그 이벤트를 처리하고, `arrayMove` 유틸리티를 사용해 배열 내 아이템 순서를 변경하는 로직을 구현해야 했다.

---

## 3. 주요 구현 내용

### 3.1 프로젝트 아키텍처
유지보수와 확장성을 고려하여 폴더 구조를 관심사별로 분리했다.
- `store/`: 전역 상태 관리 (Zustand)
- `components/`: UI 컴포넌트 (KanbanBoard, ColumnContainer, TaskCard)
- `types/`: 공통 인터페이스 정의 (TypeScript)

### 3.2 핵심 기능 구현
1.  **컬럼 및 태스크 관리**:
    - Zustand 스토어 내에 `addColumn`, `deleteColumn`, `addTask`, `deleteTask` 등의 액션을 정의하여 데이터 조작을 캡슐화했다.
2.  **Drag and Drop 로직**:
    - **Sensors**: 마우스 및 터치 이벤트를 감지하기 위해 `PointerSensor`를 설정하고, 드래그가 너무 쉽게 시작되지 않도록 10px 이동 시 활성화되게 제약 조건을 걸었다.
    - **Event Handlers**: `onDragStart`, `onDragOver`, `onDragEnd` 핸들러를 구현하여, 태스크가 다른 컬럼으로 이동하거나 같은 컬럼 내에서 순서가 바뀔 때 UI가 자연스럽게 업데이트되도록 처리했다.

### 3.3 UX 개선: 커스텀 태스크 입력
초기 구현에서는 "Add Task" 버튼 클릭 시 'Task 1', 'Task 2'와 같은 임시 제목이 자동 생성되도록 했으나, 이는 실제 사용성에 맞지 않는다고 판단했다.
- **개선**: 버튼 클릭 시 `Input` 필드가 나타나도록 변경하고, 사용자가 직접 내용을 입력한 후 Enter 키를 눌러 저장하는 방식으로 UX를 개선했다.

---

## 4. 트러블 슈팅 (Troubleshooting)

개발 과정에서 마주친 주요 기술적 문제와 해결 과정을 정리한다.

### 4.1 TypeScript "Value as Type" Import 에러
- **문제 상황**: 브라우저 콘솔에 `does not provide an export named 'Column'` 에러가 발생하며 화면이 렌더링되지 않음.
- **원인 분석**: `import { Column } from './types'` 구문을 사용할 때, TypeScript 컴파일러(Vite/esbuild)가 인터페이스를 실제 존재하는 값(Value)으로 착각하여 런타임에 찾으려 했으나, 인터페이스는 컴파일 후 사라지므로 발생한 문제다.
- **해결**: `import type { Column } ...` 구문을 사용하여 해당 참조가 타입 전용임을 명시함으로써 해결했다.

### 4.2 Tailwind CSS 버전 호환성 문제
- **문제 상황**: 스타일이 전혀 적용되지 않고, PostCSS 관련 에러 메시지가 출력됨.
- **원인 분석**: 프로젝트 설정 파일(`tailwind.config.js`)은 v3 문법을 따르고 있었으나, 패키지 설치 시 최신 버전인 v4(alpha)가 설치되어 설정 파일과 호환되지 않는 문제가 발생했다.
- **해결**: `npm install tailwindcss@3.4.17` 명령어를 통해 v3 안정화 버전으로 다운그레이드하고 서버를 재시작하여 정상화했다.

### 4.3 빌드 에러 및 실수 복구
- **문제 상황**: 태스크 입력 기능 구현 중, 실수로 드래그 앤 드롭 관련 Hook 코드 블록을 삭제하여 다량의 'Cannot find name' 에러 발생.
- **해결**: 침착하게 코드를 리뷰하여 삭제된 `useSortable` 훅과 스타일 정의 코드를 복구하고, 새로운 입력 로직과 통합했다.

---

## 5. 향후 계획
- **태스크 수정 기능**: 이미 생성된 태스크의 내용을 클릭하여 수정하는 기능 추가.
- **UI 폴리싱**: 드래그 시 애니메이션을 더 부드럽게 다듬고, 모바일 환경에서의 반응형 레이아웃 점검.
