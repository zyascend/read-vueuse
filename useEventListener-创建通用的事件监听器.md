# useEventListener-创建通用的事件监听器

文档/源码：[useEventListener | VueUse](https://vueuse.org/core/useEventListener/)

主要源码解读：

```javascript
  const stopWatch = watch(
    // 监听target的值，不管target是不是ref
    () => unref(target),
    (el) => {
      // 若target变化，包括target变为null、变为其他非null值 则先移除上次的监听
      cleanup()
      if (!el)
        return

      el.addEventListener(event, listener, options)

      cleanup = () => {
        el.removeEventListener(event, listener, options)
        // cleanup 还原为默认：() => {}
        cleanup = noop
      }
    },
    // [immediate: true] 在watch中声明的时候，就立即执行handler
    // [flush: 'post'] 决定handler执行的时机：pre在组件更新之前/post在组件更新完成之后/sync同步
    { immediate: true, flush: 'post' },
  )

  const stop = () => {
    stopWatch() // 主动调用表示停止监听
    cleanup()
  }

  // 在执行停止的时候，若有状态scope正在改变中 那么等待状态scope停止后执行
  // https://v3.vuejs.org/api/effect-scope.html#effectscope
  // https://vueuse.org/shared/tryOnScopeDispose/
  tryOnScopeDispose(stop)

  // 返回停止函数 可以主动调用移除监听
  return stop
}
```

