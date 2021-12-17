# useStorage & createGlobalState

### useStorage

**状态存入localStorage**

文档/源码：[useStorage | VueUse](https://vueuse.org/core/useStorage/)

关键代码：

1. 首先是调用`read()`尝试从localStorage中读取数据，读取与写入都经过序列化处理

   ```javascript
   function read(event?: StorageEvent) {
       if (!storage || (event && event.key !== key))
         return
       try {
         // 尝试读取
         const rawValue = event ? event.newValue : storage.getItem(key)
         if (rawValue == null) {
           data.value = rawInit
           // 如果没读到数据 尝试写入
           if (writeDefaults && rawInit !== null)
             storage.setItem(key, serializer.write(rawInit))
         }
         else {
           // 如果读到数据 为data赋值
           data.value = serializer.read(rawValue)
         }
       }
       catch (e) {
         onError(e)
       }
     }
   ```

2. 监听`storage`事件 保证`data.value`是最新的值

   ```javascript
   if (window && listenToStorageChanges)
       useEventListener(window, 'storage', e => setTimeout(() => read(e), 0))
   ```

3. 监听`data`的值，若变化，更新`storage`

   ```javascript
     if (storage) {
       watchWithFilter(
         data,
         () => {
           try {
             if (data.value == null)
               storage.removeItem(key)
             else
               storage.setItem(key, serializer.write(data.value))
           }
           catch (e) {
             onError(e)
           }
         },
         {
           flush,
           deep,
           eventFilter,
         },
       )
     }
   ```

   ### createGlobalState

   将状态`state`保存在全局的`scope`里面，可以跨vue实例重用

   使用时，先构造一个`stateFactory`

   ```javascript
   const useState = createGlobalState(() =>
     useStorage('vue-use-locale-storage', {
       name: 'Banana',
       color: 'Yellow',
       size: 'Medium',
     }),
   )
   ```

   关键代码：

   ```javascript
     let initialized = false
     let state: T
     const scope = effectScope(true)
   
     return () => {
       if (!initialized) {
         // 利用 Effect Scope API https://v3.vuejs.org/api/effect-scope.html
         state = scope.run(stateFactory)!
         initialized = true
       }
       return state
     }
   ```

   

   

