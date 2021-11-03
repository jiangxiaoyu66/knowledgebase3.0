

## 避免中间组件不必要渲染

1. 跳过中间组件传值（redux，context）
2. 通过useMemo和useCallback来提供稳定的props值
3. 子组件按需加载

## 避免不必要的渲染

只对对props进行浅对比（React.memo()）；对state和props进行前对比（React.PureConponent）

