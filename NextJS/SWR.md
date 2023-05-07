# API
```js
const { data, error, isLoading, isValidating, mutate } = useSWR(key, fetcher, options)
```
# Global Configuration
```js
<SWRConfig  value={options}>
	<Component/>
</SWRConfig>
```
Dùng để provide `options` cho tất cả SWR hooks.
## Nesting Configurations
`SWRConfig` merges config từ parent context. Value của nó có thể là `value` hoặc là `function` nhận vào config của parent và return config mới.
> Object Configuration & Functional Configuration
## Extra APIs
### Cache Provider
```js
<SWRConfig  value={{ provider: () =>  new  Map() }}>
	<Dashboard />
</SWRConfig>
```
### Access To Global Configurations
Để lấy global config ta dùng `useSWRConfig()` hook.
# Data Fetching
```js
const { data,  error } =  useSWR(key, fetcher)
```
`key`: là unique key cho mỗi request.
`fetcher`: là Promise trả về function để fetch data.
# Automatic Revalidate
## Revalidate on Focus
[Revalidate on focus](https://raw.githubusercontent.com/vercel/swr-site/master/.github/videos/focus-revalidate.mp4)
`revalidateOnFocus`
## Revalidate on Interval
```js
useSWR('/api/todos', fetcher, { refreshInterval: 1000 })
```
Tự động refetch data sau một interval và chỉ refetch khi component gắn với hook đó đang **on screen**.
## Revalidate on Reconnect
`revalidateOnReconnect`
## Disable Automatic Revalidations
Dùng `useSWRImmutable` hook để đánh dấu resource là immutable:
```js
import useSWRImmutable from  'swr/immutable'
// ...
useSWRImmutable(key, fetcher, options)

// equivalent to
useSWR(key, fetcher, { revalidateIfStale:  false, revalidateOnFocus:  false, revalidateOnReconnect:  false})
```
# Arguments
Mặc định `key` được truyền vào fetcher dưới dạng arguments.
```js
useSWR('/api/user', () => fetcher('/api/user'))
useSWR('/api/user', url => fetcher(url))
useSWR('/api/user', fetcher)
```
## Multiple Arguments
Trong một số trường hợp, ta muốn pass nhiều args vào `fetcher`, VD:
```js
useSWR('/api/user', url => fetchWithToken(url, token))
```
Tuy nhiên cách trên là ***không đúng***. Vì `key` được cache là `/api/user` nên khi token thay đổi thì SWR vẫn return về data cũ. Thay vì đó ta có thể sử dụng **array** làm key.
```js
const { data: user } = useSWR(['/api/user', token], ([url, token]) => fetchWithToken(url, token))
```
## Passing Objects
```js
const { data: orders } = useSWR({ url: '/api/orders', args: user }, fetcher)
```
# Mutation & Revalidation
## `mutate`
Có 2 cách sử dụng `mutate` API:
### Global Mutate
```js
import { useSWRConfig } from "swr"
 
function App() {
  const { mutate } = useSWRConfig()
  mutate(key, data, options)
}
```
> Lưu ý sử dụng global mutator mà chỉ truyền `key` vào thì sẽ không update cache hay trigger revalidation trừ khi có mounted SWR sử dụng chung key.
### Bound Mutate
```js
const { data,  mutate } =  useSWR('/api/user', fetcher)

mutate({ ...data, name: newName }) // không cần key
```
`mutate` này được bound với key truyền vào `/api/user`
### Revalidation
Khi gọi `mutate(key)` hoặc `mutate()` mà không truyền data vào thì nó sẽ trigger revalidate (mark data là expired và trigger refetch).
```js
mutate('/api/user') // sẽ bắt tất cả các SWRs dùng key này phải revalidate
```
### Return Values
```js
try {
  const user = await mutate('/api/user', updateUser(newUser))
} catch (error) {
  // Handle an error while updating the user here
}
```
## `useSWRMutation`
```js
import useSWRMutation from 'swr/mutation'

async function updateUser(url, { arg }: { arg: string }) {
  await fetch(url, {
    method: 'POST',
    headers: {
      Authorization: `Bearer ${arg}`
    }
  })
}
 
function Profile() {
  // A useSWR + mutate like API, but it will not start the request automatically.
  const { trigger } = useSWRMutation('/api/user', updateUser, options?)
 
  return <button onClick={() => {
    // Trigger `updateUser` with a specific argument.
    trigger('my_token')
  }}>Update User</button>
}
```
## Optimistic Updates
Là technique update UI ngay lập tức khi gửi request trước khi nhận response về từ server (update data locally) để tạo cảm giác responsive.
```js
import useSWR, { useSWRConfig } from 'swr'
 
function Profile () {
  const { mutate } = useSWRConfig()
  const { data } = useSWR('/api/user', fetcher)
 
  return (
    <div>
      <h1>My name is {data.name}.</h1>
      <button onClick={async () => {
        const newName = data.name.toUpperCase()
        const user = { ...data, name: newName }
        const options = {
          optimisticData: user,
          rollbackOnError(error) {
            // If it's timeout abort error, don't rollback
            return error.name !== 'AbortError'
          },
        }
 
        // updates the local data immediately
        // send a request to update the data
        // triggers a revalidation (refetch) to make sure our local data is correct
        mutate('/api/user', updateFn(user), options);
      }}>Uppercase my name!</button>
    </div>
  )
}
```
`optimisticData` cũng có thể là function với arg là current data.
```js
trigger(newName, { 
			optimisticData: user => ({ ...user, name: newName }),
			rollbackOnError:  true 
		})
```
Sử dụng `rollbackOnError` để rollback khi remote mutation fail.
## Update Cache After Mutation
Nếu mutation trả về updated data thì ta không cần làm thêm một extra fetch request mà chỉ cần bật `populateCache` để update cache useSWR với response đó.
```js
const updateTodo = () => fetch('/api/todos/1', {
  method: 'PATCH',
  body: JSON.stringify({ completed: true })
})
 
mutate('/api/todos', updateTodo, {
  populateCache: (updatedTodo, todos) => {
    // filter the list, and return it with the updated item
    const filteredTodos = todos.filter(todo => todo.id !== '1')
    return [...filteredTodos, updatedTodo]
  },
  // Since the API already gives us the updated information,
  // we don't need to revalidate here.
  revalidate: false
})
```
## Avoid Race Condition
`mutate` và `useSWRMutation` có thể avoid race condition với `useSWR`.
## Mutate Based on Current Data
Khi ta muốn update 1 phần của data dựa theo current data sử dụng `mutate`, ta có thể pass `async function` có arg là cached value và return updated document.
```js
mutate('/api/todos', async todos => {
  // let's update the todo with ID `1` to be completed,
  // this API returns the updated data
  const updatedTodo = await fetch('/api/todos/1', {
    method: 'PATCH',
    body: JSON.stringify({ completed: true })
  })
 
  // filter the list, and return it with the updated item
  const filteredTodos = todos.filter(todo => todo.id !== '1')
  return [...filteredTodos, updatedTodo]
// Since the API already gives us the updated information,
// we don't need to revalidate here.
}, { revalidate: false })
```
## Mutate Multiple Items
Global mutator nhận vào 1 filter function, nhận key là tham số đầu vào và trả về giá trị boolean (giống như filter của Array)
```js
import { mutate } from 'swr'
// Or from the hook if you customized the cache provider:
// { mutate } = useSWRConfig()
 
mutate(
  key => typeof key === 'string' && key.startsWith('/api/item?id='),
  undefined,
  { revalidate: true }
)
```
Có thể dùng với key là array:
```js
useSWR(['item', 123], ...)
useSWR(['item', 124], ...)
useSWR(['item', 125], ...)
 
mutate(
  key => Array.isArray(key) && key[0] === 'item',
  undefined,
  { revalidate: false }
)
```
Hoặc clear tất cả cache
```js
const clearCache = () => mutate(
  () => true,
  undefined,
  { revalidate: false }
)
 
// ...clear cache on logout
clearCache()
```
# Error Handling
Nếu lỗi throw ra trong fetcher thì hook sẽ return về `error`.
## Status Code & Error Object
```js
const fetcher = async url => {
  const res = await fetch(url)
 
  // If the status code is not in the range 200-299,
  // we still try to parse and throw it.
  if (!res.ok) {
    const error = new Error('An error occurred while fetching the data.')
    // Attach extra info to the error object.
    error.info = await res.json()
    error.status = res.status
    throw error
  }
 
  return res.json()
}
 
// ...
const { data, error } = useSWR('/api/user', fetcher)
// error.info === {
//   message: "You are not authorized to access this resource.",
//   documentation_url: "..."
// }
// error.status === 403
```
## Error Retry
SWR sử dụng *exponential backoff algorithm* để retry request khi xảy ra lỗi cho phép app recover từ error một cách nhanh chóng nhưng không gây hao tốn tài nguyên.
```js
useSWR('/api/user', fetcher, {
  onErrorRetry: (error, key, config, revalidate, { retryCount }) => {
    // Never retry on 404.
    if (error.status === 404) return
 
    // Never retry for a specific key.
    if (key === '/api/user') return
 
    // Only retry up to 10 times.
    if (retryCount >= 10) return
 
    // Retry after 5 seconds.
    setTimeout(() => revalidate({ retryCount }), 5000)
  }
})
```
## Global Error Report
```js
<SWRConfig value={{
  onError: (error, key) => {
    if (error.status !== 403 && error.status !== 404) {
      // We can send the error to Sentry,
      // or show a notification UI.
    }
  }
}}>
  <MyApp />
</SWRConfig>
```
# Conditional Fetching
## Conditional
Dùng `null` hoặc pass function vào `key` để fetch data có điều kiện. Nếu function throw hoặc return falsy value thì SWR sẽ không start request.
```js
// conditionally fetch
const { data } = useSWR(shouldFetch ? '/api/data' : null, fetcher)
 
// ...or return a falsy value
const { data } = useSWR(() => shouldFetch ? '/api/data' : null, fetcher)
 
// ...or throw an error when user.id is not defined
const { data } = useSWR(() => '/api/data?uid=' + user.id, fetcher)
```
## Dependent
SWR cho phép ta fetch data phụ thuộc vào data khác.
# Pagination
Sử dụng `useSWRInfinite`.
## When to use `useSWR`
### Pagination
Nếu build pagination thì ta nên dùng `useSWR`:
```js
function App () {
  const [pageIndex, setPageIndex] = useState(0);
 
  // The API URL includes the page index, which is a React state.
  const { data } = useSWR(`/api/data?page=${pageIndex}`, fetcher);
 
  // ... handle loading and error states
 
  return <div>
    {data.map(item => <div key={item.id}>{item.name}</div>)}
    <button onClick={() => setPageIndex(pageIndex - 1)}>Previous</button>
    <button onClick={() => setPageIndex(pageIndex + 1)}>Next</button>
  </div>
}
```
### Infinite Loading
## `useSWRInfinite`
# Subscription
## `useSWRSubscription`
Là hook cho phép ta subscribe tới real-time data source với SWR.
```js
useSWRSubscription<Data, Error>(key: Key, subscribe: (key: Key, options: { next: (error?: Error | null, data: Data) => void }) => () => void): { data?: Data, error?: Error }
```
Hook này tự động update và trả về data mới nhất khi nhận được event.
```js
function subscribe(key, { next }) {
  const sub = remote.subscribe(key, (err, data) => next(err, data))
  return () => sub.close()
}
```
Có thể được dùng để subscribe tới Firestore hay Websocket.
# Prefetching Data
## Top-Level Page Data
Có nhiều các để prefetch data cho SWR. Top level request thì recommend dùng `rel="reload"`
```js
<link rel="preload" href="/api/data" as="fetch" crossorigin="anonymous">
```
## Programmatically Prefetch
```js
import { useState } from 'react'
import useSWR, { preload } from 'swr'

const fetcher = (url) => fetch(url).then((res) => res.json())
 
// Preload the resource before rendering the User component below,
// this prevents potential waterfalls in your application.
// You can also start preloading when hovering the button or link, too.
preload('/api/user', fetcher)
 
function User() {
  const { data } = useSWR('/api/user', fetcher)
  ...
}
 
export default function App() {
  const [show, setShow] = useState(false)
  return (
    <div>
      <button onClick={() => setShow(true)}>Show User</button>
      {show ? <User /> : null}
    </div>
  )
}
```
## Pre-fill Data
Nếu ta muốn prefill existing data vào SWR cache thì dùng `fallbackData` options. Nếu SWR chưa fetch data thì nó sẽ return prefetchData làm fallback.
```js
useSWR('/api/data', fetcher, { fallbackData: prefetchedData })
```
# Usage with Next.js
# Middleware
## Usage
Là feature cho phép ta execute logic trước và sau SWR hooks. Nếu như có nhiều middleware thì mỗi middleware sẽ wrap middleware tiếp sau nó, middleware cuối cùng trong list sẽ nhận original SWR hook `useSWR`.
> Note: middleware name nên đặt theo camelCase.
```js
function myMiddleware (useSWRNext) {
  return (key, fetcher, config) => {
    // Before hook runs...
 
    // Handle the next middleware, or the `useSWR` hook if this is the last one.
    const swr = useSWRNext(key, fetcher, config)
 
    // After hook runs...
    return swr
  }
}
```
```js
<SWRConfig value={{ use: [myMiddleware] }}>
// or...
useSWR(key, fetcher, { use: [myMiddleware] })
```
## Extend
```js
function Bar () {
  useSWR(key, fetcher, { use: [c] })
  // ...
}
 
function Foo() {
  return (
    <SWRConfig value={{ use: [a] }}>
      <SWRConfig value={{ use: [b] }}>
        <Bar/>
      </SWRConfig>
    </SWRConfig>
  )
}
```
Tương đương (vì tính extends của config - Nesting Configurations)
```js
useSWR(key, fetcher, { use: [a, b, c] })
```
## Multiple Middleware
```js
useSWR(key, fetcher, { use: [a, b, c] })
```
```
enter a
  enter b
    enter c
      useSWR()
    exit  c
  exit  b
exit  a
```
