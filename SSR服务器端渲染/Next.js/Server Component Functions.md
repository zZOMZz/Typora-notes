[toc]

# Server Component Functions

## 一. fetch

- `fetch(url, options)`

- 可以直接在Server Component中直接利用`await/async`调用`fetch`, 并且默认是会有缓存(除了dynamic data fetch)

- Next.js通过原生的 [Web `fetch()` API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API), 允许服务器上的每个请求设置自己的持久化缓存语义
- 在浏览器中, cache选项指示fetch请求如何与浏览器的HTTP缓存交互. 通过这个扩展, cache选项指示服务器端(server-side)端的fetch请求如何与框架的持久化HTTP缓存交互

```js
export default async function Page() {
  // This request should be cached until manually invalidated.
  // Similar to `getStaticProps`.
  // `force-cache` is the default and can be omitted.
  const staticData = await fetch(`https://...`, { cache: 'force-cache' });

  // This request should be refetched on every request.
  // Similar to `getServerSideProps`.
  const dynamicData = await fetch(`https://...`, { cache: 'no-store' });

  // This request should be cached with a lifetime of 10 seconds.
  // Similar to `getStaticProps` with the `revalidate` option.
  const revalidatedData = await fetch(`https://...`, {
    next: { revalidate: 10 },
  });

  return <div>...</div>;
}
```

