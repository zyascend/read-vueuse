# useDraggable

**使元素可以拖动**

文档/源码：[useDraggable | VueUse](https://vueuse.org/core/useDraggable)

关键代码：

```javascript
if (isClient) {
    // 监听【按下】【移动】【抬起】三个事件
    useEventListener(target, 'pointerdown', start, true)
    useEventListener(draggingElement, 'pointermove', move, true)
    useEventListener(draggingElement, 'pointerup', end, true)
  }
...
if (isClient) {
    // 监听【按下】【移动】【抬起】三个事件
    useEventListener(target, 'pointerdown', start, true)
    useEventListener(draggingElement, 'pointermove', move, true)
    useEventListener(draggingElement, 'pointerup', end, true)
  }
...
// 【按下】事件 确定坐标零点pressedDelta
const rect = unref(target)!.getBoundingClientRect()
const pos = {
    x: e.pageX - rect.left,
    y: e.pageY - rect.top,
}
...
// 【拖动】事件。pageX - pressedDelta 得到相对于浏览器窗口的left，top
position.value = {
    x: e.pageX - pressedDelta.value.x,
    y: e.pageY - pressedDelta.value.y,
}
...
return {
    // 返回推拽时的数据
    ...toRefs(position),
    position,
    isDragging: computed(() => !!pressedDelta.value),
    style: computed(() => `left:${position.value.x}px;top:${position.value.y}px;`),
  }
```

相关坐标位置：

![useDraggable01](https://github.com/zyascend/read-vueuse/blob/main/pics/useDraggable01.png)
