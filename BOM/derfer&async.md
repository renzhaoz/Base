# defer & async

## 简介

同步javascript脚本会阻塞html解析。defer和sync则会优化改阻塞。
script标签中的defer属性和sync属性的用法和差别。

## 特点

- 改变脚本在dom树中的执行时机,减少js和dom解析之间的阻塞时间
- defer属性的脚本会在dom解析完成之后,DOMContentLoaded事件之前执行
- async属性的脚本会在下载完成时执行,有可能在DOMContentLoaded之前，也有可能在之后,但肯定在load事件之前。如果html正在解析时async属性的js加载完了  那个会停止解析html去执行js 完成后再解析html
- defer属性的的脚本不会阻塞html的解析，等待html解析完成后才会执行。执行完成后才会触发DOMContentLoaded。有defer属性的脚本会严格按照在dom中的先后顺序执行
- 一个脚本中两个属性都存在时defer失效
- defer和async只对外部脚本有效

## 起步API

        <script defer src='path'></script>
        <script defer src='path'></script>

## 拓展

- async 立即下载脚本 不阻塞下载资源 或等待其它脚本 只对外部脚本有效
- defer 脚本被延迟到文档完全被解析和显示之后再执行 只对外部脚本有效
