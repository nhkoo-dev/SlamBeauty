# SlamBeauty

SlamBeauty는 K-뷰티 브랜드와 마이크로/미드티어 인플루언서를 연결하는 캠페인 매칭 플랫폼입니다.

브랜드는 캠페인을 생성하고 인플루언서를 매칭·관리하며, 인플루언서는 캠페인에 지원하고 콘텐츠 진행 상황과 정산 내역을 확인합니다. 현재는 프로토타입 단계로, 백엔드 없이 `lib/dummy-data.ts`의 목데이터로 전체 화면 흐름을 구현하고 있습니다.

---

## Tech Stack

- Next.js 16 (App Router)
- React 19
- TypeScript
- Tailwind CSS 4
- lucide-react
- class-variance-authority / clsx / tailwind-merge

> `AGENTS.md`에 명시된 대로 이 프로젝트의 Next.js 16은 기존에 알려진 컨벤션과 다른 부분(breaking changes)이 있습니다. 코드 작성 전 `node_modules/next/dist/docs/`의 가이드를 확인하세요.

---

## Project Structure

```
app/
├── layout.tsx                          # 루트 레이아웃 (Geist 폰트, 메타데이터, lang="ko")
├── page.tsx                             # 랜딩 페이지 ('/', 브랜드·인플루언서 진입점 분기)
├── globals.css
│
├── brand/                                # 브랜드(광고주) 페이지 그룹
│   ├── layout.tsx                        #   BrandSidebar + 콘텐츠 영역 공통 레이아웃 (인증 가드 없음)
│   ├── login/page.tsx                    #   로그인 폼(비연동) + 데모 체험 버튼 → /brand/dashboard
│   ├── dashboard/page.tsx                #   KPI 카드, 예산 진행률, 대기 액션, 캠페인 현황 요약
│   ├── campaigns/page.tsx                #   전체 캠페인 목록 (상태 탭 UI, 인플루언서 파이프라인 카운트)
│   ├── campaigns/[id]/page.tsx           #   캠페인 상세: 콘텐츠 검수 · 인플루언서 파이프라인 진행 · 지원자 승인/반려 · 예산 내역
│   ├── campaign/new/page.tsx             #   캠페인 생성 3단계 마법사 (상품정보 → 타겟팅 → 가이드라인/예산/일정)
│   ├── matching/page.tsx                 #   인플루언서 탐색·매칭 (검색/필터/정렬, 협업 요청)
│   ├── analytics/page.tsx                #   성과 대시보드 (도달/참여/지출/CPE, 캠페인별·카테고리별 비교)
│   └── settings/page.tsx                 #   브랜드 프로필 · 타겟팅 기본값 · 알림 · 청구 설정 (로컬 상태만)
│
├── influencer/                           # 인플루언서 페이지 그룹
│   ├── layout.tsx                        #   로그인/온보딩 페이지는 사이드바 없이, 나머지는 InfluencerSidebar로 감쌈
│   ├── login/page.tsx                    #   로그인 버튼 → useInfluencerAuth().login() → /influencer/campaigns
│   ├── onboarding/page.tsx               #   프로필 온보딩 3단계 (SNS·기본정보 → 단가표 → 약관 동의)
│   ├── dashboard/page.tsx                #   /influencer/campaigns로 즉시 리다이렉트
│   ├── active/page.tsx                   #   진행 중 캠페인 파이프라인 카드 (트래킹, 가이드라인 체크리스트, 콘텐츠 링크 제출)
│   ├── campaigns/page.tsx                #   공개 캠페인 마켓플레이스 (비로그인도 열람 가능, 지원은 로그인 필요)
│   ├── campaigns/[id]/page.tsx           #   캠페인 상세 (브랜드 정보, 가이드라인, 정산 내역, 경쟁률) + 지원 모달
│   ├── profile/page.tsx                  #   마이페이지 (프로필 수정 + 지원/진행/완료 내역 탭)
│   └── settlement/page.tsx               #   정산/출금 페이지 (수수료 내역, 계좌 정보, 출금 요청)
│
components/
├── ui/                                    # shadcn 스타일 공용 프리미티브 (수동 구현, CLI 생성 아님)
│   ├── avatar.tsx / badge.tsx / button.tsx / card.tsx / input.tsx
├── brand/
│   ├── brand-sidebar.tsx                 #   브랜드 좌측 네비게이션 (대기 액션 배지 포함)
│   └── influencer-detail-sheet.tsx       #   인플루언서 상세 슬라이드 시트 (프로필 열람 / 지원자 승인·반려)
└── influencer/
    └── influencer-sidebar.tsx            #   인플루언서 좌측 네비게이션 (비로그인 시 잠금 아이콘 + 로그인 유도)

lib/
├── dummy-data.ts                          # 전체 앱을 구동하는 목데이터 (인플루언서/캠페인/정산/프로필 등)
├── use-influencer-auth.ts                # localStorage 기반 인플루언서 로그인·프로필완료 상태 훅
└── utils.ts                               # cn() 클래스 병합 유틸
```

`app/brand`와 `app/influencer`는 완전히 독립된 두 축으로, 각자의 `layout.tsx`가 자신의 사이드바를 감쌉니다. 두 축이 공유하는 것은 `components/ui`의 공용 프리미티브와 `lib/`의 목데이터·유틸뿐입니다.

---

## Features

### Home (`/`)

- 브랜드사 / 인플루언서 두 갈래로 분기하는 마케팅 랜딩 페이지

### Brand (`/brand/*`)

- 로그인(데모 체험) → 대시보드에서 캠페인 현황·예산·대기 액션 확인
- 캠페인 생성(3단계 마법사), 목록/상세 조회
- 캠페인 상세에서 지원자 승인·반려, 인플루언서 파이프라인(승인→배송→수령→콘텐츠 제출→최종 승인→정산) 진행 관리, 제출 콘텐츠 검수
- 인플루언서 탐색·매칭 및 협업 요청
- 캠페인 성과 애널리틱스, 브랜드 설정

### Influencer (`/influencer/*`)

- Google 로그인 버튼(목업) → 온보딩(SNS·기본정보, 단가표, 약관 동의)
- 공개 캠페인 마켓플레이스 열람(비로그인 가능) 및 지원(로그인 필요)
- 진행 중 캠페인의 파이프라인 관리(트래킹 정보 입력, 가이드라인 체크, 콘텐츠 링크 제출)
- 마이페이지(프로필 수정, 지원/진행/완료 내역), 정산 내역 조회 및 출금 요청

---

## Architecture

```
Next.js (App Router)
      │
      ▼
Page/Client 컴포넌트 (useState)
      │
      ▼
lib/dummy-data.ts (정적 목데이터)
```

현재 백엔드·DB·API 라우트는 없으며, 모든 화면은 `lib/dummy-data.ts`의 목데이터를 각 페이지의 로컬 상태로 읽어와 렌더링합니다. 새로고침 시 상태는 초기화되며, 유일한 영속 상태는 `use-influencer-auth.ts`가 사용하는 `localStorage`(로그인 여부·프로필 완료 여부)뿐입니다. `zustand`가 의존성으로 설치되어 있지만 아직 어디에서도 사용되지 않습니다.

---

## Getting Started

```bash
npm install
npm run dev
```

[http://localhost:3000](http://localhost:3000)에서 확인할 수 있습니다.
