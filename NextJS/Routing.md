# Fundamentals

## app directory

- Component bên trong _app_ là React Server Component.

## Folders and Files inside _app_

**Folder** dùng để define route

**File** được dùng để tạo UI cho route segment.

## Route Segments

Mỗi folder trong route đại diện cho 1 _route segment_ và map tới 1 segment tương ứng trong URL path.

## Nested Routes

Nested folder = Nested route

## File Conventions

Các file đặc biệt tương ứng với các behaviour trong nested route:

- **page.js**: UI cho route và giúp cho route có thể access.
  - **route.js**: Server-side API endpoint cho route.
- **layout.js**: Shared UI cho segment và children.
  - **template.js**: 
- **loading.js**: Wrap page hoặc child segment trong *React Suspense Boundary* và hiện loading UI khi chúng đang load.
- **error.js**: Tương tự loading.js nhưng wrap trong *Error Boundary*
  - **global-error.js**:Tương tự error.js nhưng catch errors trong root *layout*
- **not-found.js**: Hiện khi hàm *notFound* được throw bên trong route segment.

## Component Hierarchy

![Component Hierarchy](https://assets.vercel.com/image/upload/f_auto,q_100,w_1600/v1675248777/nextjs-docs/darkmode/file-conventions-component-hierarchy.png)

![Component Hierarchy](https://assets.vercel.com/image/upload/f_auto,q_100,w_1600/v1675248778/nextjs-docs/darkmode/nested-file-conventions-component-hierarchy.png)

## Colocation

Bên cạnh các file đặc biệt thì ta có thể thêm các file của mình vào folder như stylesheets, tests, components,...

## Server-Centric Routing with Client-side Navigation

Router trong app dùng **server-centric routing** tận dụng *Server Component* và *data fetching bên phía server* 

​	=> Client không cần download route map. Tuy vậy router vẫn dùng *client-side nav* với *Link Component* 

​	=> Browser không reload khi navigate tới route khác mà chỉ update URL và rerender segment đã thay đổi.

Router sẽ lưu result của RSC payload vào **in-memory client-side catch**.

## Partial Rendering

Chỉ refetch hay rerender layout và page trong route segment thay đổi chứ không ảnh hưởng đến những segment nằm bên trên trong subtree.

## Advance Routing Patterns

1. Parallel Routes
2. Intercepting Routes
3. Conditional Routes

# Defining Routes

## Creating Routes

Trong app, ta dùng folders để define route. Mỗi folder đại diện 1 *route segment* tương ứng với URL segment. Nested route = Nested folder

![Route segment](https://assets.vercel.com/image/upload/f_auto,q_100,w_1600/v1667553431/nextjs-docs/darkmode/route-segments-to-path-segments.png)

## Creating UI

Trong route segment folder phải có **page.js** để làm cho route đó accessible.

![Page files make route segments publicly accessible](https://assets.vercel.com/image/upload/f_auto,q_100,w_1600/v1666568301/nextjs-docs/darkmode/defining-routes-page.js.png)

## Route Groups

Tạo **route groups** để:

- Tổ chức folder mà không ảnh hưởng tới URL structure.
- Sử dụng các layout khác nhau cho cùng route segment.
- Tạo nhiều root layout bằng cách chia nhỏ app.

### Convention

**(folderName)**

![Route group](https://assets.vercel.com/image/upload/f_auto,q_100,w_1600/v1666568349/nextjs-docs/darkmode/route-group-organisation.png)

![Multiple layout](https://assets.vercel.com/image/upload/f_auto,q_100,w_1600/v1666568349/nextjs-docs/darkmode/route-group-multiple-layouts.png)

### Good to know

- `(marketing)/about/page.js` và `(shop)/about/page.js` cùng trỏ tới `/about` và sẽ gây ra lỗi.

- Multiple root layout sẽ gây ra **full page load** khi navigate giữa các layout.

## Dynamic Segment

Nếu ta không biết trước được route segment và muốn fill nó sau tại `request time` hoặc prerendered lúc `build time`

Sử dụng tên folder như sau: **[folderName]**. Dynamic segments sẽ được truyền vào `params` props cho layout, page, route và `generateMetadata` function.

```js
export default function Page({ params }) {
  return <div>My Post</div>;
}

/* app/blog/[slug]/page.js - /blog/a - params: { slug: 'a' } */
```

### Catch-all segments

**[...folderName]**: sẽ match với: `/shop/clothes`, `shop/clothes/tops` và `shop/clothes/tops/t-shirt`.

### Optional Catch-all

**[[...folderName]]**: Tương tự Catch-all nhưng sẽ match với cả `/shop` nữa.

# Pages and Layouts

## Pages

![Page.js](https://assets.vercel.com/image/upload/f_auto,q_100,w_1600/v1667559940/nextjs-docs/darkmode/page.png)

```js
// `app/page.js` is the UI for the root `/` URL
export default function Page() {
  return <h1>Hello, Next.js!</h1>;
}
```

- Page luôn là *leaf* trong route subtree.
- Mặc định là Server Component nhưng có thể đổi sang Client Component.
- Pages có thể fetch data.

## Layouts

Layout được share giữa nhiều page. Khi navigate thì state được giữa nguyên, vẫn interactive và không rerender. Layout cũng có thể nested.

`default export` từ file layout.js, component nhận vào `children`.

``` javascript
export default function DashboardLayout({
  children, // will be a page or nested layout
}: {
  children: React.ReactNode,
}) {
  return (
    <section>
      {/* Include shared UI here e.g. a header or sidebar */}
      <nav></nav>

      {children}
    </section>
  );
}
```

- Top most layout = Root layout. Phải chứa `html` và `body`
- Route segment có thể define layout riêng và layout này sẽ được dùng chung cho tất cả các page trong segment đó.
- Layout trong route mặc định được nested ~ layout cha wrap layout con nằm dưới nó.
- Layout là SC nhưng có thể set thành CC.
- Layout có thể fetch data.
- Không thể truyền data từ layout vào children của nó nhưng có thể fetch cùng data ở trong cùng route nhiều lần nhờ cơ chế dedupe request.

### Root Layout

- Là bắt buộc phải có và được defined tại top level của app directory.

- Có thể dùng `route groups` để tạo nhiều root layouts.

- Là SC và **không** thể set thành CC được. 

![Nested layout](https://assets.vercel.com/image/upload/f_auto,q_100,w_1600/v1666568352/nextjs-docs/darkmode/nested-layout.png)

![Nested layout](https://assets.vercel.com/image/upload/f_auto,q_100,w_1600/v1671641894/nextjs-docs/darkmode/nested-layouts.png)

## Modifying `<head>`

Chúng ta có thể thay đổi `<head>` như `title` hay `meta` bằng cách export `metadata` object hoặc `generateMetadata` function trong layout.js hoặc page.js.

# Linking and Navigating

NextJS router dùng **server-centric routing** và **client-side navigation*:

- Support instant loading states & concurent rendering.
- Navigation duy trì được client-side state, tránh rerender, không bị gián đoạn.

2 cách để navigate:

- `<Link>` component.
- `useRouter` hook.

## Link Component

`<Link>` component extends HTML `<a>` element + *prefetching* và *client-side navigation*. Là cách chính để navigate giữa các Route trong NextJS.

```js
import Link from 'next/link';

export default function Page() {
  return <Link href="/dashboard">Dashboard</Link>;
}
```

## useRouter() hook

```js
'use client';

import { useRouter } from 'next/navigation';

export default function Page() {
  const router = useRouter();

  return (
    <button type="button" onClick={() => router.push('/dashboard')}>
      Dashboard
    </button>
  );
}
```

`useRouter` còn cung cấp các methods như `push()` hay `refresh()`,...

## How Navigation Works

1. Route transition bắt đầu từ `<Link>` hoặc gọi `router.push()`.
2. Router update URL trong browser address bar.
3. Router tránh unnecessary work bằng cách tái sử dụng các segment nào không thay đổi (e.g. shared layout) từ **client-side cached**. ~ Partial rendering.
4. Nếu điều kiện *soft navigation* đáp ứng thì router fetch segment mới từ cached thay vì gọi server. Còn không thì thực hiện *hard navigation* và fetch SC payload từ server.
5. Khi xảy ra thì loading UI sẽ được hiện payload đang được fetch.
6. Router sử dụng *cached* hoặc *fresh payload* để render segment mới cho client.

### Client-side Caching of Rendered SC

> Client-side cache khác với server-side NextJS HTTP cache.

Cơ chế router mới có 1 **in-memory client-side cache** lưu trữ rendered result từ SC (payload). Cache được chia thành route segments cho phép chúng ta invalidate tại các level khác nhau.

Một số trường hợp router sẽ re-use cache thay vì tạo request mới lên server giúp cải thiện performance.

### Invalidating the cache

Sử dụng `router.refresh()`.

### Prefetching

Là cơ chế preload 1 route trong background trước khi nó được truy cập. Rendered result của prefetched routes sẽ được add vào client-side cache của router giúp trải nghiệm navigate gần như ngay lập tức.

Mặc định: route được prefetch khi `Link` component visible trong viewport. Ta cũng có thể dùng `prefetch` của `useRouter` hook.

**Static and Dynamic Routes**:

- Route là static thì SC payloads của route segment sẽ được prefetched.
- Route là dynamic thì payload của *first shared layout down until the first loading.js file* sẽ được prefetched. Giúp giảm cost prefetch toàn bộ dynamic route.

Prefetching chỉ enabled trong production và có thể được disabled bằng cách truyền `prefetch={false}` vào `<Link>`.

### Hard Navigation

Cache bị invalidate và server refetch data cũng như re-renders segment đã thay đổi khi navigate.

### Soft Navigation

Cache cho segment đó được tái sử dụng (nếu có) và không tạo request mới lên server khi navigate.

Điều kiện:

- Soft navigation khi navigate đến route đã được prefetched và (không chứa *dynamic segment* hoặc có cùng dynamic parameter với route hiện tại.)

  > `dashboard/team-red/*` tới `dashboard/team-red/*` sẽ là soft navigation.
  >
  > `dashboard/team-red/*` tới `dashboard/team-blue/*` sẽ là hard navigation.

- Back/Forward navigation là soft navigation.

# Loading UI

![Loading.js](https://assets.vercel.com/image/upload/f_auto,q_100,w_1600/v1666704760/nextjs-docs/darkmode/loading-ui.png)

## Instant Loading States

Là fallback UI hiện ngay lập tức khi navigate. Chúng ta có thể pre-render loading indicator như skeletons hay spinners để giúp người dùng biết là app vẫn đang responding.

![Loading file](https://assets.vercel.com/image/upload/f_auto,q_100,w_1600/v1673552977/nextjs-docs/darkmode/loading-folder.png)

![Structure](https://assets.vercel.com/image/upload/f_auto,q_100,w_1600/v1666568376/nextjs-docs/darkmode/loading-diagram.png)

- Navigation là immediate kể cả với *server-centric routing*.
- Navigation là interruptible ~ không cần đợi content của route fully load trước khi navigate sang route khác.
- Shared layout vẫn interactive khi new route segment load.

# Error Handling

Handle runtime errors trong nested routes:

- Wrap route segment và nested children bên trong `Error Boundary`.
- Chỉ segment bị lỗi mới bị ảnh hưởng, các phần còn lại của app vẫn hoạt động bình thường.
- Recover từ error không cần full page reload.

![Error.js](https://assets.vercel.com/image/upload/f_auto,q_100,w_1600/v1673552977/nextjs-docs/darkmode/error-folder.png)

![How error.js works](https://assets.vercel.com/image/upload/f_auto,q_100,w_1600/v1666568378/nextjs-docs/darkmode/error-diagram.png)

## Recovering From Errors

Lý do gây ra lỗi đôi khi có thể là temporary chỉ cần retry là giải quyết được. Ta có thể dụng `reset()` để giúp người dùng recover from error. Khi được gọi, hàm này sẽ rerender Error Boundary content và nếu thành công thì fallback error sẽ được thay thế bởi result của lần rerender.

```js
'use client';

export default function Error({
  error,
  reset,
}: {
  error: Error;
  reset: () => void;
}) {
  return (
    <div>
      <h2>Something went wrong!</h2>
      <button onClick={() => reset()}>Try again</button>
    </div>
  );
}
```

## Nested Routes

![](https://assets.vercel.com/image/upload/f_auto,q_100,w_1600/v1673552978/nextjs-docs/darkmode/nested-error-component-hierarchy.png)

## Handling Errors in Layouts

`error.js` không catch error throw ra từ `layout.js` hay `template.js` từ cùng segment.

=> Để thêm error.js lên segment ở level bên trên.

## Handling Errors in Root Layouts

Nếu muốn catch error của root layout hoặc template thì dùng `global-error.js`. Vì nó wrap cả app nên nó sẽ replace root layout khi active => cần có thẻ `html` và `body` của riêng nó.

# Route Handlers

## Convention

Route handler được defined trong `app/route.js` 

```js
export async function GET(request: Request) {}
```

### Supported HTTP Methods

`GET`, `POST`, `PUT`, `PATCH`, `DELETE`, `HEAD`, và `OPTIONS`

### Extended `NextRequest` và `NextResponse` APIs

## Behavior

### Static Route Handlers

Route handler mặc định được *statically evaluated*  khi dùng `GET` với `Response` object.

```js
import { NextResponse } from 'next/server';

export async function GET() {
  const res = await fetch('https://data.mongodb-api.com/...', {
    headers: {
      'Content-Type': 'application/json',
      'API-Key': process.env.DATA_API_KEY,
    },
  });
  const data = await res.json();

  return NextResponse.json({ data })
}
```

### Dynamic Route Handlers

Route handlers được evaluted dynamically khi:

- Dùng `Request` object với `GET` method.
- Dùng các HTTP methods khác.
- Dùng *Dynamic Functions* như `cookies` hay `headers`.

> Không thể có route.js và page.js ở cùng route => `app/page.js` và `app/route.js` là **conflict** nhưng `app/page.js` và `app/api/route.js` thì **valid**.

# Parallel Routes

Cho phép ta render nhiều page đồng thời trong cùng 1 view nhưng vẫn có thể navigate tới từng page độc lập. Với những dynamic sections của app như dashboard và social site, Parallel Routes có thể được dùng để implement complex routing patterns.

## Convention

Được defined với folder có ttên **@folder** và được gọi là **slot**.

```txt
dashboard
├── @audience
│   ├── demographics
│   │   └── page.js
│   ├── subscribers
│   │   └── page.js
│   └── page.js
├── @views
│   ├── impressions
│   │   └── page.js
│   ├── view-duration
│   │   └── page.js
│   └── page.js
├── layout.js
└── page.js
```

`dashboard/layout.js` nhận các slot `@audience` và `@views` vào từ props và render chúng song song bên cạnh children props.

```js
function AudienceNav() {
  return <nav>...</nav>;
}

function ViewsNav() {
  return <nav>...</nav>;
}

export default function Layout({ children, audience, views }) {
  return (
    <>
      <h1>Tab Bar Layout</h1>
      {children}

      <h2>Audience</h2>
      <AudienceNav />
      {audience}

      <h2>Views</h2>
      <ViewsNav />
      {views}
    </>
  );
}
```

- `children` ngầm định là 1 slot => `dashboard/page.js` ~ `dashboard/@children/page.js`.

## Behavior

### URL

Slot không ảnh hướng đến URL structure (tương tự route groups).

# Intercepting Routes





