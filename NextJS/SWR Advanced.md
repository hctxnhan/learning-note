# Understanding SWR
## State Machine
- `isValidating`: `true` bất cứ khi nào có request được gửi đi kể cả data đã load hay chưa.
- `isLoading`: `true` bất cứ khi nào có request được gửi đi khi data chưa được load.
```js
function Stock() {
  const { data, isLoading, isValidating } = useSWR(STOCK_API, fetcher, {
    refreshInterval: 3000
  });
 
  // If it's still loading the initial data, there is nothing to display.
  // We return a skeleton here.
  if (isLoading) return <div className="skeleton" />;
 
  // Otherwise, display the data and a spinner that indicates a background
  // revalidation.
  return (
    <>
      <div>${data}</div>
      {isValidating ? <div className="spinner" /> : null}
    </>
  );
}
```
## Return previous data for better UX
`keepPreviousData` 
## Dependency Collection for performance
SWR chỉ trigger rendering khi states sử dụng trong component update => nếu chỉ dùng data thì `isValidating` và `isLoading` thay đổi không trigger update => reduce rendering => increase performance.

