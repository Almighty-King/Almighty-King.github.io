
---
title: Px2Rem Webpack 插件初试
---

> 在h5开发中，我们做设备兼容经常会用到 rem，通常会有个问题就是 rem 和设计稿 px 之间的转换。正好为了学习一下 Webpack 插件的开发，所以就尝试了下简单写个 Webpack 插件来完成这个转换。

# 实现逻辑

> 遍历项目中的 css 文件 正则匹配所有的 px 单位，然后根据传入的换算比例计算为对应的 rem。

# 代码

```js
class Px2RemPlugin {
  constructor(options) {
    this.options = options;
  }

  apply(compiler) {
    // 在 emit 阶段进行处理，emit 是文件生成资源到 output 目录之前执行的钩子，也是能修改资源的最后时机
    compiler.hooks.emit.tapAsync('Px2RemPlugin', (compilation, callback) => {
      const { remUnit } = this.options; // rem 单位
      const { assets } = compilation;
      // 遍历所有资源文件
      Object.entries(assets).forEach(([filename, source]) => {
        if (filename.endsWith('.css')) {
          const content = source.source();
          const newContent = content.replace(/(\d+)px/g, (match, p1) => {
            const pxValue = parseInt(p1, 10);
            const remValue = pxValue / remUnit;
            return `${remValue}rem`;
          });
          // 将修改后的内容替换回资源文件
          compilation.assets[filename] = {
            source: () => newContent,
            size: () => newContent.length,
          };
        }
      });
      callback();
    });
  }
}

module.exports = Px2RemPlugin;
```

# 引入

```js
module.exports = {
  // ... 其他配置
  plugins: [new Px2RemPlugin({ remUnit: 75 })],
};
```
