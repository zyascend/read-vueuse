# useClipboard

**使用剪贴板**

文档/源码：[[useClipboard | VueUse](https://vueuse.org/core/useClipboard/#type-declarations)](https://vueuse.org/core/useEventListener/)

相关资料：

主要使用`navigator.clipboard`

[Clipboard API - Web APIs | MDN (mozilla.org)](https://developer.mozilla.org/en-US/docs/Web/API/Clipboard_API)

[剪贴板操作 Clipboard API](https://www.ruanyifeng.com/blog/2021/01/clipboard-api.html)

主要源码解读：

```javascript
function updateText() {
    // @ts-expect-error untyped API
    navigator.clipboard.readText().then((value) => {
      text.value = value
    })
  }

  if (isSupported && read) {
    for (const event of events)
      // 实时监听系统的拷贝、剪贴事件 方便及时更新text值
      useEventListener(event as WindowEventName, updateText)
  }

  async function copy(value = unref(source)) {
    if (isSupported && value != null) {
      // 操作是异步的，返回Promise对象，不会造成页面卡顿
      // @ts-expect-error untyped API
      await navigator.clipboard.writeText(value)
      text.value = value
      copied.value = true
      // 开始写入计时 超过copiedDuring时间，copied=false
      timeout.start()
    }
  }

  return {
    // 如果navigator.clipboard属性返回undefined，就说明当前浏览器不支持这个API
    isSupported,
    // 当前剪贴板里的文字内容，可响应式获取最新的值
    text: text as ComputedRef<string>,
    // 是否写入成功写入剪贴板
    copied: copied as ComputedRef<boolean>,
    // 提供主动写入剪贴板的方法 方便后续使用
    copy,
  }
```

备注：

> 使用时需要明确获得用户的许可。权限的具体实现使用了 Permissions API，跟剪贴板相关的有两个权限：`clipboard-write`（写权限）和`clipboard-read`（读权限）。"写权限"自动授予脚本，而"读权限"必须用户明确同意给予。也就是说，写入剪贴板，脚本可以自动完成，但是读取剪贴板时，浏览器会弹出一个对话框，询问用户是否同意读取。
>
> https://www.ruanyifeng.com/blog/2021/01/clipboard-api.html

