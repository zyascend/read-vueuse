# useFetch

**封装fetch()方法**

文档/源码：[useFetch | VueUse](https://vueuse.org/core/useFetch/#basic-usage)

#### 1. 当Url改变时如何自动重新发起fetch

```javascript
  watch(
    () => [
      // 1. 监听url的变化 触发execute()
      unref(url),
      // 2. options.refetch由false变为true时 触发execute()
      unref(options.refetch),
    ],
    () => unref(options.refetch) && execute(),
    { deep: true },
  )
```

#### 2. 避免request在创建时立即触发

```javascript
if (options.immediate)
    // immediate==false时 只会返回execute 不会执行execute
    setTimeout(execute, 0)
```

#### 3. 中断请求

##### 3.1 确定 `canAbort`

```javascript
// 通过判断当前浏览器是否支持AbortController接口确定supportsAbort
const supportsAbort = typeof AbortController === 'function'
// canAbort为true的条件 supportsAbort && isFetching
const canAbort = computed(() => supportsAbort && isFetching.value)
```

##### 3.2 执行中断`abort()`

```javascript
if (supportsAbort) {
      // 首先创建AbortController对象
      controller = new AbortController()
      // abort之后设置aborted为true
      controller.signal.onabort = () => aborted.value = true
      fetchOptions = {
        ...fetchOptions,
        // 应用controller.signal于fetch过程中
        signal: controller.signal,
      }
    }
...
const abort = () => {
    if (supportsAbort && controller)
      // 判断满足条件：支持abort && controller，执行abort
      controller.abort()
  }
...
if (timeout)
    // 如果设置了timeout 超过时间将会abort
    timer = useTimeoutFn(abort, timeout, { immediate: false })
```

##### 3.3 劫持请求（Intercepting a request）

```javascript
if (options.beforeFetch)
      // 在assign context时, 会执行beforeFetch() 
      Object.assign(context, await options.beforeFetch(context))

// 在更新前对返回的数据进行预处理
if (options.afterFetch)
      ({ data: responseData } = await options.afterFetch({ data: responseData, response: fetchResponse }))

// 更新之前预处理
if (options.onFetchError)
      ({ data: responseData, error: errorData } = await options.onFetchError({ data: responseData, error: fetchError }))
 // 更新操作
data.value = responseData
error.value = errorData
```

