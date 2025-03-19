---
title: Vue3 在两处设置别名的原因
date: 2025-03-19 21:52:19
tags:
---
最近在读博客[脚手架的升级与配置 | Vue3 入门指南与实战案例](https://vue3.chengpeiquan.com/upgrade.html#%E7%AE%A1%E7%90%86%E9%A1%B9%E7%9B%AE%E9%85%8D%E7%BD%AE-1)

别名有着简化路径引用的功能，可以防止写出`../../../../components/HelloWorld.vue`的引用。

在调整TS Config这里有点不懂，为什么要在`vite.config.ts`和`tsconfig.json`中各自配置别名。

其中 `vite.config.ts`中配置方法是

```typescript
export default defineConfig({
  // ...
  resolve: {
    alias: {
      '@': resolve('src'), // 源码根目录
      '@img': resolve('src/assets/img'), // 图片
      '@less': resolve('src/assets/less'), // 预处理器
      '@libs': resolve('src/libs'), // 本地库
      '@plugins': resolve('src/plugins'), // 本地插件
      '@cp': resolve('src/components'), // 公共组件
      '@views': resolve('src/views'), // 路由组件
    },
  },
  // ...
})
```

`tsconfig.json`的方法是：

```typescript
{
  "compilerOptions": {
    // ...
    "paths": {
      "@/*": ["src/*"],
      "@img/*": ["src/assets/img/*"],
      "@less/*": ["src/assets/less/*"],
      "@libs/*": ["src/libs/*"],
      "@plugins/*": ["src/plugins/*"],
      "@cp/*": ["src/components/*"],
      "@views/*": ["src/views/*"]
    },
    // ...
  },
  // ...
}
```

刚开始颇为不解，经过查证发现，其实`vite.config.ts`中配置的是构建工具的配置文件，而`tsconfig.json`中配置的是TypeScript的配置文件，主要是两处文件的负责范围的不同。

1. Vite在运行时或者编译阶段负责文件路径的解析，代码运行时，构建工具要明确路径别名和实际目录的对应关系。

因此，构建工具的别名主要作用于：

* 模块打包（编译时）
* 文件寻址（运行时）

2. TypeScript在开发阶段会尝试解析代码中的模块路径，因此TypeScript也是需要知道路径别名和实际文件之间的映射关系的，如果TypeScript不认识别名就会提示找不到模块的警告

因此，TypeScript的别名主要作用于：

* 代码开发（静态类型检查）
* 编辑器提示（路径的补全和跳转）

总的来说，两处没有关联，但是又各自有需要路径别名的需求，因此都需要设置一下。
