# useActiveElement

**获取文档中当前获得焦点的元素**

文档/源码：[useActiveElement | VueUse](https://vueuse.org/core/useActiveElement/)

主要源码解读：

```javascript
export function useActiveElement<T extends HTMLElement>(options: ConfigurableWindow = {}) {
  const { window = defaultWindow } = options
  const counter = ref(0)

  if (window) {
    // 在window上监听blur和focus事件，触发counter值的改变
    useEventListener(window, 'blur', () => counter.value += 1, true)
    useEventListener(window, 'focus', () => counter.value += 1, true)
  }

  return computed(() => {
    // eslint-disable-next-line no-unused-expressions
    counter.value // 这个computed依赖counter的值，一旦其改变，返回最新的活动元素
    // document.activeElement属性返回文档中当前获得焦点的元素
    return window?.document.activeElement as T | null | undefined
  })
}

```

