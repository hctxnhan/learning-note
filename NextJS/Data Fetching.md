# Fundamentals

## `fetch()` API
Là hệ thống data fetching mới được built on top `fetch() Web API` để tận dụng `async`/`await` trong SC.
- **Automatic request deduping**
- Extend `fetch` options cho phép mỗi request tự set rule về caching và revalidating.

## Fetching Data on the Server

SC luôn fetch data nằm trên server, cho phép ta:

- Direct access to BE resource.
- Tránh lộ thông tin nhạy cảm như *token* hay *API keys*.
- Fetch data và render trên cùng 1 environment.
- Reduce client-server *waterfall*.

## Component-level Data Fetching

Ta có thể fetch data trong *page*, *layout*, và *component*. Vì ta không thể truyền data giữa parent layout với children của nó nên ta có thể fetch data ở đâu nó cần. React và NextJS sẽ tự động cache và dedupe requests để tránh data được fetch nhiều lần.

### Parallel and Sequential Data Fetching

![](https://assets.vercel.com/image/upload/f_auto,q_100,w_1600/v1669651658/nextjs-docs/darkmode/sequential-parallel-data-fetching.png)

### Automatic `fetch()` Request Deduping

![](https://assets.vercel.com/image/upload/f_auto,q_100,w_1600/v1673006076/nextjs-docs/darkmode/deduplicated-fetch-requests.png)

- Optimization áp dụng cho các `fetch` request xảy ra trong Layouts, Pages, SC, `generateMetadata` và `generateStaticParams`. 
- Lifetime của cache là cho đến khi rendering process hoàn thành.
- `POST` request không tự động dedupe.

## Static and Dynamic Data Fetches

![](https://assets.vercel.com/image/upload/f_auto,q_100,w_1600/v1669662198/nextjs-docs/darkmode/dynamic-and-static-data-fetching.png)

Trogn quá trình render, khi Nextjs gặp fetch request thì nó sẽ check cache trước để xem có data sẵn hay không. Nếu có thì return cached data còn không thì fetch và lưu data vào cache.

### Revalidating Data

- **Background**: Revalidates theo một time interval.
- **On-demand**: Revalidates khi có update.

### Streaming and Suspense

---

# Fetching

## `async`/`await` trong SC

```js
export default async function Page() {
  const data = await getData(); // ẢO MA CHƯA CU

  return <main></main>;
}
```

## SC functions

- `cookies()`
- `headers()`

## `use` in CC

`use` vẫn đang còn experimental nên ta có thể dùng **SWR** hoặc **React Query** thay thế.

## Static Data Fetching

Mặc định fetch sẽ cache data internally.

```js
fetch('https://...'); // cache: 'force-cache' is the default
```

### Revalidating Data

```js
fetch('https://...', { next: { revalidate: 10 } });
```

## Data Fetching Patterns

### Parallel Data Fetching

Dùng `Promise.all`.

### Sequentil Data Fetching

### Blocking Rendering in a Route

Nếu fetch data trong layout thì tất cả route segment nằm dưới nó chỉ có thể bắt đầu khi data đã load xong. => **all or nothing** data fetching, ta có thể giải quyết bằng cách:

- Sử dụng `loading.js`.
- Di chuyển data fetching xuống thấp hơn trong component tree => chỉ block rendering cho phần nào cần data đó.

> Best practice: Fetch data trong segment nào cần nó thôi.

## Data Fetching without `fetch()`

Khi ta không dùng `fetch` được vì dùng third-party lib như ORM hay db client mà vẫn muốn control caching hay revalidating behavior của layout và page thì có thể phụ thuộc vào *default caching behavior* của segment hoặc *segment cache configuration*.

### Default Caching Behavior

Data fetching libraries không dùng fetch trực tiếp thì sẽ không affect caching của route, nó là dynamic hay static phụ thuộc hoàn toàn vào route segment.

### Segment Cache Configuration

Ta có thể customize cache behavior cho toàn bộ segment.

```js
import prisma from './lib/prisma';

export const revalidate = 3600; // revalidate every hour

async function getPosts() {
  const posts = await prisma.post.findMany();
  return posts;
}

export default async function Page() {
  const posts = await getPosts();
  // ...
}
```

---

# Caching

## Segment-level Caching

Cho phép ta cache và revalidate data sử dụng trong route segments.

```js
export const revalidate = 60; // revalidate this page every 60 seconds
```

Ngoài ta ta có thể dùng `fetch` cùng với `revalidate` option để revalidation route theo interval:

```js
// revalidate this page every 10 seconds, since the getData's fetch
// request has `revalidate: 10`.
async function getData() {
  const res = await fetch('https://...', { next: { revalidate: 10 } });
  return res.json();
}

export default async function Page() {
  const data = await getData();
  // ...
}
```

Nếu page, layout và fetch cùng specify revalidation frequency thì value nào thấp nhất sẽ được chọn.

## Per-request Caching

Những function nào ta không thể control được (third party lib chẳng hạn), ta có thể wrap nó bằng `cache()` để có được per-request caching behvior.

### Preload

preload là một pattern chứ không phải API.

```js
import { getUser } from "@utils/getUser";

export const preload = (id: string) => {
  // void evaluates the given expression and returns undefined
  // https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/void
  void getUser(id);
}
export default async function User({ id }: { id: string }) {
  const result = await getUser(id);
  // ...
}
```

```js
import User, { preload } from '@components/User';

export default async function Page({
  params: { id },
}: {
  params: { id: string };
}) {
  preload(id); // starting loading the user data now
  const condition = await fetchCondition();
  return condition ? <User id={id} /> : null;
}
```

### Combining `cache`, `preload` and `server-only`

```js
import { cache } from 'react';
import 'server-only';

export const preload = (id: string) => {
  void getUser(id);
}

export const getUser = cache(async (id: string) => {
  // ...
});
```

Với cách tiếp cận này ta có thể:

- Eagerly fetch data.
- Cache responses.
- Đảm bảo data fetching chỉ xảy ra trên server.

---

# Revalidating

Giúp ta update specific static route mà không cần rebuild toàn bộ site. Revalidation aka. *Incremental Static Regeneration*.

## Background Revalidation

- Dùng `next.revalidate` option trong `fetch`.
- Dùng route segment config.
- Dùng `cache`

### How it works

VD ta set revalidate = 60s:

1. Khi request tới route đã được statically rendered at build time, ban đầu nó sẽ show cached data.
2. Các request tới route trước 60s cũng sẽ nhận về cached.
3. Sau 60s, request đầu tiên vẫn nhận cached (stale) data.
4. Next.js trigger regeneration cho data trong background.
5. Khi route đã generate xong thì Next sẽ invalidate cache và show updated route. Nếu regeneration fails thì data cũ vẫn không bị thay thế.

## On-demand Revalidation

### Using On-Demand Revalidation

1. Đầu tiên ta tạo secret token cho Next.js app. Sau đó access route với URL struture như sau:

```bash
https://<your-site.com>/api/revalidate?secret=<token>
```

2. Thêm secret vào *Environment Variable*
3. Tạo revalidation API Route (trong `pages` directory):

```js
export default async function handler(req, res) {
  // Check for secret to confirm this is a valid request
  if (req.query.secret !== process.env.MY_SECRET_TOKEN) {
    return res.status(401).json({ message: 'Invalid token' });
  }

  try {
    // This should be the actual path not a rewritten path
    // e.g. for "/blog/[slug]" this should be "/blog/post-1"
    await res.revalidate('/path-to-revalidate');
    return res.json({ revalidated: true });
  } catch (err) {
    // If there was an error, Next.js will continue
    // to show the last successfully generated page
    return res.status(500).send('Error revalidating');
  }
}
```

## Error Handling and Revalidation

If an error is thrown while attempting to revalidate, the last successfully generated route will continue to be used. On the next subsequent request, Next.js will retry revalidating.

---

# Mutating

> Nextjs vẫn đang implement cơ chế mutation data mới sử dụng RFC và chưa được publish



# Streaming and Suspense

## What is Streaming?

SSR và những hạn chế của nó.

Với SSR, có một số việc phải hoàn thành trước khi người dùng có thể thấy và tương tác với page.

1. Fetch tất cả data cần cho page.
2. Render thành HTML.
3. Bundle HTML, CSS, JS cho page và gửi tới client.
4. Non-interactive UI được show (*chỉ có HTML và CSS*).
5. React hydrates UI để làm nó interactive.

![](https://assets.vercel.com/image/upload/f_auto,q_100,w_1600/v1671640463/nextjs-docs/darkmode/ssr-chart.png)

SSR giúp cải thiện loading performance bằng cách show non-interactive page tới user càng sớm càng tốt. Nhưng nó vẫn cần phải đợi data fetching bên phía server hoàn thành => vẫn chậm.

**Streaming** cho phép ta chia nhỏ page HTML thành nhiều mảnh và từ từ gửi từng mảnh về client.

| ![](https://assets.vercel.com/image/upload/f_auto,q_100,w_1600/v1666630985/nextjs-docs/darkmode/server-rendering-without-streaming.png) | ![](https://assets.vercel.com/image/upload/f_auto,q_100,w_1600/v1666631081/nextjs-docs/darkmode/server-rendering-with-streaming.png) |
| ------------------------------------------------------------ | ------------------------------------------------------------ |

Streaming làm việc tốt với React Component vì mỗi component có thể xem là 1 chunk. 

- Component có **priority cao hơn** hoặc **không phụ thuộc vào data** sẽ được gửi đi trước => React có thể hydrate nó sớm hơn.
- Component có **priority thấp hơn** có thể được gửi sau (trong cùng 1 request) khi data của chúng đã được fetched.

![](https://assets.vercel.com/image/upload/f_auto,q_100,w_1600/v1671640286/nextjs-docs/darkmode/server-rendering-with-streaming-chart.png)

## Streaming in Next.js

Implement streaming trong Next.js sử dụng `loading.js` cho toàn bộ route segment hoặc *Suspense boundary* (**required**).

Nếu dùng Suspense, ta có được các benefit sau:

1. Streaming SR.
2. Selective Hydration.

---

# Generate Static Params

`generateStaticParams` là function có thể dùng kèm với *dynamic route segment* để statically generate routes tại build time thay vì on-demand request time.

```js
export async function generateStaticParams() {
  const posts = await fetch('https://.../posts').then((res) => res.json());

  return posts.map((post) => ({
    slug: post.slug,
  }));
}
```

> Cần phải xem lại.

# API Routes

> Đã được thay thế bởi Route Handlers trong Next.js 13.2.

