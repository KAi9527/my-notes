useState              ~= 带 UI 自动刷新的成员变量
useEffect             ~= 生命周期回调 / 定时任务 / 事件监听
useRef                ~= 普通可变字段，不触发 UI 更新
useMemo               ~= 缓存一个计算值
useCallback           ~= 缓存一个函数对象
useParams             ~= 从请求路径里取 path variable
useSnapshot           ~= 订阅全局状态
自定义 useXxx          ~= 复用一段组件状态逻辑的工具方法