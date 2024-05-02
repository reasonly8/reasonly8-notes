# reasonly8-notes

My personal technical notes.
这是我的个人技术文档。

## 背景

长久以来，我都在想办法对抗遗忘，无论是我生命中各种美好的瞬间，还是学习时积累的各种知识，我都想尽可能“记住”，因为遗忘意味着重新开始，意味着“折线行走”。

想要在有限的生命中，走得更远，必须得有个好记性。

这个项目就是为了试图解决这一问题而存在的，虽然不能完全解决，但它“延缓”遗忘的效果还是挺好的。

为什么写笔记能延缓遗忘？我的理解是

它通过 Github Workflow 自动构建，然后发布到 VS Code 插件市场，这样我就可以只专注于写，而不用在意部署，并且阅读也很方便，打开 vscode 就能看。

##

这份笔记，与其说是笔记，不如说是项目文档，因为每篇笔记，都是一个完整的项目。
笔记中记录的，是在实现这个项目时，遇到的一些“值得”记录的知识点，或是注意事项。
在我看来，项目，就是解决某个问题的“方案”，虽然这个“方案”大多是通过写代码实现的，但绝不仅限于此。
一个想法，从萌生到最终实现，期间用到的各种技术（工具）、遇到的各种问题，以及解决这些问题的方法，都是值得记录的。
虽然记录会花时间，但比起未来因为遗忘而重复劳动来说，是很划算的。
而且，这样记笔记还能将一组弱相关的知识紧密结合起来，有助于加深对各个知识点的理解。
项目的开发和笔记的撰写同步进行，两者都不是一蹴而就的，都是在写和改的循环中螺旋推进，笔记可以帮助理清项目实现的思路，项目又能让笔记有血有肉，更有价值。
每篇笔记都适合作为学习资料，因为它是完整的，因为完整，所以能快速建立起对陌生技术的直观印象，它解决了什么问题，要使用什么工具，中途可能遇到什么坑，最后怎样使用，如果重写，哪些地方可以改进等等。

```
reasonly8-notes
├─ 📁.github
│  └─ 📁workflows
│     └─ 📄update_vscode_extension.yml
├─ 📁git
│  └─ 📄修改远程仓库地址.md
├─ 📁github-api
│  └─ 📄获取仓库压缩包并解压到指定目录.md
├─ 📁reasonly8-notes-vscode-extension
│  └─ 📄关于.md
├─ 📁reasonly8.github.io
│  ├─ 📄DNS 轮询.md
│  ├─ 📄GitHub Pages 使用步骤.md
│  ├─ 📄关于.md
│  ├─ 📄清除本机 DNS 缓存.md
│  └─ 📄理解 DNS 解析中的 TTL 参数.md
├─ 📁secret-npm-pkg
│  ├─ 📄ESM 下获取当前脚本所在路径.md
│  ├─ 📄git tag 的基础用法.md
│  ├─ 📄npm publish 发布带作用域前缀的包.md
│  ├─ 📄npm version 用法.md
│  ├─ 📄package.json 中 type 字段的作用.md
│  ├─ 📄tsconfig.json 中 moduleResolution 的作用.md
│  ├─ 📄tsconfig.json 中 target 和 module 的区别.md
│  ├─ 📄关于.md
│  ├─ 📄在 package.json 中声明 bin 字段以安装命令行工具.md
│  ├─ 📄用 crypto 进行 RSA 加密和解密.md
│  ├─ 📄获取汉字的字节长度.md
│  └─ 📄设置 import 类型时必须加 type.md
├─ 📁ssh
│  ├─ 📄生成 SSH RSA 密钥对.md
│  ├─ 📄配置多个 SSH 密钥对.md
│  └─ 📄配置远程主机免密登录.md
├─ 📁tree-node-order
│  └─ 📄关于.md
├─ 📁TypeScript
│  └─ 📁Tuple
│     └─ 📄只读元组.md
├─ 📁vscode-extension
│  ├─ 📄初始化插件项目.md
│  ├─ 📄发布一个 VS Code Extension.md
│  ├─ 📄在 Activity Bar 中添加图标.md
│  ├─ 📄弹出右下角提示信息.md
│  ├─ 📄打开 Activity Bar 中的某个 Extension.md
│  └─ 📄注册一个 Command Palette 中的命令.md
├─ 📄README.md
└─ 📄todo-list-for-learning-vitest.md
```