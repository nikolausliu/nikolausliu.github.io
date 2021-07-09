---
title: eslint的无法识别ts全局type解决方案
date: 2021-06-28 21:41:47
tags:
  - TypeScript
  - ESLint
categories:
comments:
---

我在ts项目中声明了一个全局type，在使用这个type的时候，可以获得类型提示，但是eslint的`no-undef`规则无法校验通过

```index.d.ts
declare global {
  type IdLike = number | string | null
}
```

<img src="/images/20210628/01.png" alt="全局type声明" width="100%" />
<img src="/images/20210628/02.png" alt="eslint no-undef" width="100%" />

查阅eslint官方文档得知，需要配置eslint配置文件（`.eslintrc.*`文件或`package.json`文件的`eslintConfig`字段）里的`globals`字段：

```
{
  globals: {
    IdLike: true
  }
}
```

声明了几个全局type就要配置几个，还挺麻烦的，但是目前没找到更好的解决方案。

我这里是用作了全局type，globals更常见的用法是设置全局变量，值可以是`writable`和`readonly`，很好理解就是字面意思，由于历史包袱这两个值分别有两个备选项，效果一致。

```
{
    "globals": {
        "var1": "writable",  // 或"writeable" 或 true
        "var2": "readonly"  // 或"off" 或 false
    }
}
```

更多参考官方文档：

- [disallow-undeclared-variables-no-undef](https://eslint.org/docs/rules/no-undef#disallow-undeclared-variables-no-undef)
- [specifying-globals](https://eslint.org/docs/user-guide/configuring/language-options#specifying-globals)