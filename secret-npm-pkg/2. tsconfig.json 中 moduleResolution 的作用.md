moduleResolution：模块解析策略，它规定了 TS 在导入模块时，以什么策略去查找这个模块，常用的有：Node、Classic、Bundler 等。

如果使用的是 TypeScript 5.0+ 版本，官方会建议使用 `Bundler`。

> 它（Bundler）的出现解决的最大痛点就是：可以让你使用 exports 声明类型的同时，使用相对路径模块可以不写扩展名。

参考文章：https://juejin.cn/post/7221551421833314360
