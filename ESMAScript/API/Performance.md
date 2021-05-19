# Performance性能监测API

## 简介

&nbsp;&nbsp;&nbsp;&nbsp;页面性能始终是 Web 开发者关心的话题。 Performance 接口通过 JavaScript API 暴露了浏览器内部
的度量指标,允许开发者直接访问这些信息并基于这些信息实现自己想要的功能。这个接口暴露在
window.performance 对象上。所有与页面相关的指标,包括已经定义和将来会定义的,都会存在于
这个对象上。

## 起步API

Performance由以下几个API组成.

- High Resolution Time API
Date.now只适用于日期的相关操作，如果用于收集耗时信息精度可能有问题，例：
 若函数执行过快，则收集的两个信息有可能相等。
 修改系统时间后该值可能是负值或者极大值。

  window.performance.now()适用于准确到微秒精度的浮点数的耗时收集。且该值只会正增长。

- Performance Timeline API
- Navigation Timing API
- User Timing API
- Resource Timing API
- Paint Timing API