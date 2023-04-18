# Fundamentals

## Rendering Environments

![Client and Server Environment](https://assets.vercel.com/image/upload/f_auto,q_100,w_1600/v1666704761/nextjs-docs/darkmode/client-and-server.png)

## *Component-level* Client and Server Rendering

Ta có thể chọn rendering environment ở component level. Mặc định app directory dùng SC cho phép chúng ta render components trên server và giảm lượng JS phải gửi về client.

Ta có thể dùng SC và CC trong component tree bằng cách import CC vào SC hoặc pass SC vào CC dưới dạng `children` hoặc `props`.

## Static and Dynamic Rendering on the Server

### Static Rendering

Với SR thì cả SC và CC đều được prerendered trên server lúc ***build time***. Kết quả sẽ được cached và tái sử dụng trong các request sau đó. Cached result cũng có thể được revalidated.

SC và CC được render khác nhau khi dùng Static Rendering:

- CC: HTML và JSON được prerendered và cached trên server sau đó được gửi về cho client để *hydration*.
- SC: được rendered trên server bởi React và payload được dùng để generate HTML. Cùng một rendered payload đó sẽ được dùng để hydrate component bên phía client => không cần JS phải chạy bên phía client.

### Dynamic Rendering

SC và CC được rendered tại ***request time*** với dynamic render và không được cached.

# Server & Client Components

> Kết hợp giữa *rich interactivity* của client-side apps và *performance* của server rendering.

## Server Components

### Why Server Components

SC cho phép developer tận dụng sức mạnh của server. VD large dependencies trước kia sẽ ảnh hưởng tới JS bundle size gửi về client giờ đây được giữ lại hoàn toàn bên server. => Improve performance.

- Initial page load: faster.
- Client-side JS bundle size: smaller.
- Additional JS chỉ được thêm vào nếu client-side interactivity cần đén nó trong CC.

## Client Components

Cho phép ta add client-side interactivity vào app. CC được Nextjs prerendered bên phía Server và hydrated bên phía Client.

### Convention

Dùng `use client` directive nếu muốn khai báo một component là CC. Đặt ở đầu file. Các module import vào nó kể cả child component cũng được tính vào client bundle.

> - Một SC được đảm bảo chỉ render bên phía server.
> - Không cần khai báo `use client` trong tất cả các file, chỉ cần define 1 lần và các module import vào nó đều được tính là CC.

![](https://assets.vercel.com/image/upload/f_auto,q_100,w_1600/v1680683516/nextjs-docs/darkmode/use-client-directive.png)

## When to use SC vs CC?

- Khi phải dùng interactive và event listener.
- `useState` hay `useEffect`
- Browser-only API

Thì dùng CC.

Còn lại dùng SC. *(fetch data, access BE resource directly, secret information, large deependencies,...)*

## Moving Client Components to the Leaves

## Importing SC into CC

Nếu làm như dưới thì SC sẽ thành CC chứ không còn là SC nữa.

```js
'use client';

// ❌ This pattern will not work. You cannot import a Server
// Component into a Client Component
import ServerComponent from './ServerComponent';

export default function ClientComponent() {
  return (
    <>
      <ServerComponent />
    </>
  );
}
```

Thay vào đó ta có thể làm như sau:

```js
// ✅ This pattern works. You can pass a Server Component
// as a child or prop of a Client Component.
import ClientComponent from "./ClientComponent";
import ServerComponent from "./ServerComponent";

// Pages are Server Components by default
export default function Page() {
  return (
    <ClientComponent>
      <ServerComponent />
    </ClientComponent>
  );
}
```

Làm như vầy sẽ giúp React biết là SC cần rendered trên server trước khi gửi về CC.

## Passing props from Server to Client Components (Serialization)

## Keeping Server-Only Code out of Client Components (Poisoning)

Cài các package `server-only` và `client-only`.

## Sharing data between SC

Vì SC không interactive nên không có React state => Không cần sức mạnh của context để share data mà có thể dùng pattern như *Singleton* để nhiều SC cùng access.

## Sharing fetch requests between Server Components

`fetch` request tự động dedupe trong SC => dùng data ở đâu thì gọi ở đó không lo về duplicate requsssests.

# Static and Dynamic Rendering

- **static** at build time.
- **dynamic** at request time.

## Static Rendering (default)

Ta có thể chuyển từ SR sang DR bằng cách sử dụng *dynamic function* hoặc *dynamic data fetching* trong layout hoặc page. Khiến NextJS render toàn bộ route dynamically tại *request time*.

| Data Fetching | Dynamic Functions | Rendering |
| ------------- | ----------------- | --------- |
| Cached        | No                | Static    |
| Cached        | Yes               | Dynamic   |
| Not Cached    | No                | Dynamic   |
| Not Cached    | Yes               | Dynamic   |

> Để một route là SR thì **không** dùng dynamic functions + data request đã được cached.

## Static Data Fetching (default)

[Caching and Revalidating](https://beta.nextjs.org/docs/data-fetching/caching)

## Dynamic Rendering

### Dynamic Functions

Dynamic functions là function phụ thuộc vào những info chỉ có thể biết được tại request time như: *user's cookies, current request headers, URL search params*. Trong Nextjs:

- `cookies()` và `header()` trong SC sẽ opt toàn bộ route thành DR tại request time.
- `useSearchParams()` trong CC sẽ skip SR và render tất cả các **CC đến Suspense boundary gần nhất** trên client. => Nên wrap các CC dùng `useSearchParams` vào trong `<Suspense/>` để những CC bên trên vẫn SR bình thường.

### Dynamic Data Fetches

Là `fetch()` mà không dùng cache bằng cách:

- Set `cache` thành `no-store`, hoặc.
- Set `revalidate` thành `0`.

