---
title: FSD 아키텍처 마이그레이션
date: 2024-11-25 19:00:00 +/-TTTT
categories: [React]
tags: [javascript, react, FSD, book-rating] # TAG names should always be lowercase
description: 기존 프로젝트에 FSD 아키텍처 적용하기
---

## 기존 폴더 구조

기존에는 따로 아키텍처라고 할 것도 없이 pages, actions, hooks 등 큰 주제별로 폴더를 구분해두었다.

1. ui **폴더의 모호성 증가**: 원래 ui는 shadcn/ui 컴포넌트가 설치되는 폴더로 시작했지만, shadcn 외에 재사용할 search 와 같은 컴포넌트가 생기면서 어떤 컴포넌트를 여기에 넣어야 할지 애매해짐.

2. **특정 페이지 전용 컴포넌트**: 특정 페이지에서만 사용하는 컴포넌트를 어디에 두어야 할지 명확한 기준이 없음.

3. **폴더 크기 증가로 인한 관리 어려움**: ui 폴더가 너무 커지면 가독성이 떨어지고, 특정 컴포넌트를 찾는 데 시간이 오래 걸림.

```
📦src
 ┣ 📂app
 ┃ ┣ 📂(other)
 ┃ ┃ ┣ 📂login
 ┃ ┃ ┣ 📂register
 ┃ ┣ 📂(books)
 ┃ ┃ ┣ 📂books
 ┃ ┃ ┃ ┣ 📂[id]
 ┃ ┃ ┃ ┣ 📂add
 ┃ ┣ 📂api
 ┃ ┃ ┗ 📂kakao
 ┃ ┃ ┃ ┗ 📂books
 ┣ 📂components
 ┃ ┣ 📂auth
 ┃ ┣ 📂book
 ┃ ┣ 📂icons
 ┃ ┣ 📂layout
 ┃ ┣ 📂review
 ┃ ┣ 📂ui
 ┃ ┃ ┣ 📜alert-dialog.tsx
 ┃ ┃ ┣ 📜avatar.tsx
 ┃ ┃ ┣ 📜badge.tsx
 ┃ ┃ ┣ 📜button.tsx
 ┃ ┃ ┣ 📜card.tsx
 ┃ ┃ ┣ 📜combobox.tsx
 ┃ ┃ ┣ 📜command.tsx
 ┃ ┃ ┣ 📜debounce-input.tsx
 ┃ ┃ ┣ 📜dialog.tsx
 ┃ ┃ ┣ 📜form.tsx
       // .... more
 ┣ 📂contexts
 ┗ 📂lib
 ┃ ┣ 📂actions
 ┃ ┣ 📂axios
 ┃ ┣ 📂hooks
 ┃ ┣ 📜types.ts
 ┃ ┗ 📜utils.ts
```

아키텍처에 대해 고민하던 중 FSD에 대해 알게되었다. **Feature-Sliced Design** (FSD)는 프론트엔드 애플리케이션의 구조를 잡는 아키텍처 방법론이다.
레이어 (최상위폴더) > 슬라이스(도메인별) > 세그먼트(코드 수행 역할별) 이런 계층 구조로 되어있다.

이미 작업중인 프로젝트에 FSD를 적용하려고 한다. [마이그레이션 가이드 문서](https://feature-sliced.design/kr/docs/guides/migration/from-custom)가 있어서 참고하였다. 총 8단계까지인데, 6단계부터는 선택이라서 5단계까지만 적용해 보려고 한다.

## 1. pages 코드 나누기

src/pages/books/ui/books-page.tsx
각 페이지에 대한 폴더를 만들고 인덱스 파일을 추가하자.

```tsx
import React from "react";
import BookList from "@/components/book/book-list";
import Link from "next/link";
import { Button } from "@/components/ui/button";
import { PlusIcon } from "@radix-ui/react-icons";

export function BooksPage() {
  return (
    <>
      <div className="container mx-auto py-8 px-4 md:px-6">
        <div className="flex items-center justify-start mb-6">
          <Button
            asChild
            variant="default"
            className="transition-transform duration-300 border shadow-md bg-primary rounded-sm font-semibold"
          >
            <Link href="/books/add">
              <PlusIcon className="w-5 h-5" />
              Add Book
            </Link>
          </Button>
        </div>
        <div className="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 xl:grid-cols-5 gap-6">
          <BookList />
        </div>
      </div>
    </>
  );
}
```

src/pages/books/index.ts

```tsx
export { BooksPage } from "./ui/books-page";
```

src/app/(main)/page.tsx

```tsx
import { BooksPage } from "@/pages/books";

export default function Home() {
  return <BooksPage />;
}
```

내 프로젝트는 라우터가 books/, books/\[id\] 이렇게 있었는데, app 라우터 폴더 구조가 중첩이 되니 pages도 동일하게 중첩되어야 하는건지 고민되었다. 하지만 [여기 Next.js FSD 예제](https://github.com/falkomerr/falkchat)를 보니 pages는 페이지별로 폴더를 만드는 것으로 보인다.

```
📦src
 ┣ 📂app
 ┃ ┃ 📂(main)
 ┃ ┃ ┣ 📂servers/[serverId]
 ┃ ┗ ┗ ┗ 📂channels/[channelId]
 ┣ 📂pages
 ┃ ┣ 📂server-id
 ┗ ┗ 📂chat-id
```

app 을 보면 servers 폴더 하위에 channels가 있지만 pages 폴더에서는 같은 레벨로 폴더가 만들어져 있다.

그래서 /books/[id]의 페이지는 @/pages/book-detail에 따로 만들었다.

```tsx
// src/app/(main)/books/[id]/page.tsx
import { BookPage } from "@/pages/book";

export default function BookDetail({
  params: { id },
}: {
  params: { id: string };
}) {
  return <BookPage id={id} />;
}
```

## 2. shared 폴더 만들고 공통 파일 이동하기

지금 구조는 이렇다

```
📦src
 ┣ 📂app
 ┣ 📂components
 ┣ 📂contexts
 ┣ 📂lib
 ┃ ┣ 📂actions
 ┃ ┣ 📂axios
 ┃ ┣ 📂hooks
 ┃ ┣ 📜types.ts
 ┃ ┗ 📜utils.ts
 ┗ 📂pages
 ┃ ┣ 📂book-add
 ┃ ┣ 📂book-detail
 ┃ ┗ 📂books
```

components에 컴포넌트들을 다 넣어놨는데 양이 많다보니 한 번에 shared로 옮기지 않고 일단 놔두려고 한다. 그래서 lib를 shared로 이름을 바꾸고 contexts는 lib 안으로 넣었다. actions는 api 호출하는 함수 모아둔건데 헷갈리니 api로 이름을 변경했다.

```
📦src
 ┣ 📂app
 ┣ 📂components
 ┣ 📂pages
 ┗ 📂shared
 ┃ ┣ 📂api
 ┃ ┣ 📂axios
 ┃ ┣ 📂contexts
 ┃ ┣ 📂hooks
```

## 3. 페이지 간의 상호 의존성 해결하기

한 페이지가 다른 페이지의 파일이나 코드를 import하는 경우를 찾아 코드를 복사하여 붙여넣거나 shared로 이동한다.
코드 성격에 따라 적절한 폴더로 이동:

- **UI 관련 코드**: shared/ui 폴더로 이동
- **상수 또는 설정값**: shared/config 폴더로 이동
- **백엔드와의 상호작용 로직**: shared/api 폴더로 이동
  가이드 문서에서 처음에 코드를 복사하라고 해서 흠칫했다. 코드 중복이 발생하잖아?? 하지만 복사 붙여넣기는 무조건 잘못된 것이 아니다고 한다. DRY 원칙(“Don’t Repeat Yourself”)에 집착해 불필요한 추상화를 도입하면 코드 유지보수가 더 어려워질 수 있다고 하니 참고하자.

책 리스트를 보여주는 페이지 컴포넌트이다.

```tsx
import React from "react";
import BookList from "@/components/book/book-list";
import Link from "next/link";
import { Button } from "@/components/ui/button";
import { PlusIcon } from "@radix-ui/react-icons";

export function BooksPage() {
  return (
    <>
      <div className="container mx-auto py-8 px-4 md:px-6">
        <div className="flex items-center justify-start mb-6">
          <Button
            asChild
            variant="default"
            className="transition-transform duration-300 border shadow-md bg-primary rounded-sm font-semibold"
          >
            <Link href="/books/add">
              <PlusIcon className="w-5 h-5" />
              Add Book
            </Link>
          </Button>
        </div>
        <div className="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 xl:grid-cols-5 gap-6">
          <BookList />
        </div>
      </div>
    </>
  );
}
```

이 페이지에서 사용하는 커스텀 컴포넌트는 BookList, Button 이다. BookList는 이 페이지에서만 사용하니 페이지 컴포넌트와 같은 경로로 이동하고, 버튼은 shared/ui로 옮겼다.

```tsx
import React from "react";
import Link from "next/link";
import { PlusIcon } from "@radix-ui/react-icons";
import BookList from "@/pages/books/ui/book-list";
import { Button } from "@/shared/ui/button";
```

이어서 BookList 안에서 사용중인 컴포넌트가 BookList 페이지에서만 사용되는거라면 마찬가지로 같은 경로로 옮기기를 반복했다.

```
📦books
 ┣ 📂ui
 ┃ ┣ 📜book-card.tsx
 ┃ ┣ 📜book-edit-modal.tsx
 ┃ ┣ 📜book-form.tsx
 ┃ ┣ 📜book-list.tsx
 ┃ ┗ 📜books-page.tsx
 ┗ 📜index.ts
```

## 4.  Shared 계층 정리

Shared에 있는 파일 중 특정 페이지에서만 사용되는 코드는 해당 페이지의 슬라이스로 옮긴다. 슬라이스는 도메인에 따라 나눠진 레이어다.

books-page에서 사용하는 BookList 컴포넌트는 fetchBooks로 책 목록을 가져온다.

```tsx
"use client";

import BookCard from "./book-card";
import { useQuery } from "@tanstack/react-query";
import { fetchBooks } from "@/shared/api/book";
import Link from "next/link";

export default function BookList() {
  const { data: books } = useQuery({
    queryKey: ["books"],
    queryFn: fetchBooks,
  });

  return (
    <>
      {books?.map((book, index) => (
        <Link href={`/books/${book.id}`} key={index}>
          <BookCard book={book} />
        </Link>
      ))}
    </>
  );
}
```

이 API는 BookList만 사용하니 이 페이지로 가져온다.

```tsx
import { useQuery } from "@tanstack/react-query";
import Link from "next/link";
import BookCard from "./book-card";
import { fetchBooks } from "../api/book";
```

## 5. 코드의 기술적 목적에 따라 조직화하기

코드를 단순히 파일 유형(components, actions, utils 등)에 따라 나누는 대신, **기술적 목적**이나 **역할**에 따라 구조화하여 코드의 의도를 더 명확하게 드러내고 유지보수성을 높인다.

FSD에서 기술적 목적에 따른 구분은 *세그먼트* 로 이루어진다 . 다음은 자주 사용되는 세그먼트들이다.

- `ui`— UI 표시와 관련된 모든 것: UI 구성 요소, 날짜 포맷터, 스타일 등
- `api`— 백엔드 상호작용: 요청 함수, 데이터 유형 등
- `model`— 데이터 모델: 스키마, 인터페이스, 저장소 및 비즈니스 로직.
- `lib`— 이 슬라이스의 다른 모듈에 필요한 라이브러리 코드.
- `config`— 구성 파일 및 기능 플래그.

지금까지 마이그레이션한 결과이다.

```
📦pages
 ┣ 📂book-add
 ┃ ┣ 📂ui
 ┃ ┃ ┗ 📜book-add-page.tsx
 ┃ ┗ 📜index.ts
 ┣ 📂book-detail
 ┃ ┣ 📂api
 ┃ ┃ ┗ 📜review.ts
 ┃ ┣ 📂model
 ┃ ┃ ┗ 📜review-interface.ts
 ┃ ┣ 📂ui
 ┃ ┃ ┣ 📜book-info.tsx
 ┃ ┃ ┣ 📜book-page.tsx
 ┃ ┃ ┣ 📜review-item.tsx
 ┃ ┃ ┗ 📜review-list.tsx
 ┃ ┗ 📜index.ts
 ┗ 📂books
 ┃ ┣ 📂api
 ┃ ┃ ┗ 📜book.ts
 ┃ ┣ 📂ui
 ┃ ┃ ┣ 📜book-card.tsx
 ┃ ┃ ┣ 📜book-edit-modal.tsx
 ┃ ┃ ┣ 📜book-form.tsx
 ┃ ┃ ┣ 📜book-list.tsx
 ┃ ┃ ┗ 📜books-page.tsx
 ┃ ┗ 📜index.ts
```

## shadcn/ui 설치 경로 변경

[shadcn/ui](https://ui.shadcn.com/)를 사용중인데, 지금은 ui 컴포넌트 설치 시 `@/componens/ui`에 설치된다. shadcn/ui는 보통 공통으로 사용될 때가 많을 것 같아서 `@/shared/ui` 에 설치되도록 `components.json` 파일에 **ui aliases**를 추가했다.

```json
{
  "$schema": "https://ui.shadcn.com/schema.json",
  "style": "default",
  "rsc": true,
  "tsx": true,
  "tailwind": {
    "config": "tailwind.config.ts",
    "css": "src/app/globals.css",
    "baseColor": "slate",
    "cssVariables": true
  },
  "aliases": {
    "components": "@/components",
    "utils": "@/shared/utils", <-- 수정
    "ui": "@/shared/ui" <-- 추가
  }
}

```

## NextJS app, pages 폴더 충돌

![라우터 폴더 충돌 에러](/assets/img/posts/2024-11-21/Pasted%20image%2020241127125846.png)
_라우터 폴더 충돌 발생_

app 라우터를 사용하는데, FSD 구조를 적용한다고 추가한 pages 하위 index 파일과 충돌이 발생했다.
[FSD 가이드 문서](https://feature-sliced.design/kr/docs/guides/tech/with-nextjs)에는 루트 경로에 app, pages 폴더를 만드는 방법이 소개되어 있다.

```
├── app                # NextJS app 폴더
├── pages              # NextJS pages 폴더
│   ├── README.md      # 해당 폴더의 목적과 역할에 대한 설명
├── src
│   ├── app            # FSD app 폴더
│   ├── entities
│   ├── features
│   ├── pages          # FSD pages 폴더
│   ├── shared
│   ├── widgets
```

루트 경로로 기존 app 폴더를 옮기고 pages 폴더를 추가한다. 추가한 pages에는 README로 설명을 적거나 [여기](https://github.com/yunglocokid/FSD-Pure-Next.js-Template/tree/master/pages)처럼 `.gitkeep`을 추가한다.
이렇게하면 src 하위에 app, pages는 FSD 폴더로 사용할 수 있다. 여기 [NextJS 에서의 사용 예시](https://stackblitz.com/edit/stackblitz-starters-aiez55?file=README.md)를 보고 동일하게 구성했더니 충돌이 해결되었다.

---

관련된 파일들이 가까이 있으니 아직은 어색하다. 실제로 개발을 해봐야지 FSD가 나의 프로젝트와 잘 맞는 아키텍처인지 알 수 있을 것 같다. 좀 더 익숙해졌을 때 마이그레이션 선택 단계인 6~8단계도 이어서 적용해보자.
