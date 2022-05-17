# React 源码阅读（序）｜写在前面&本地 build

不知哪刻热血上头，信誓旦旦，想拿下`React`源码，遂激动地打开[源码](https://github.com/facebook/react)，待翻阅片刻，思绪平定，不禁自问，源码浩瀚，从何开始？思索无果，兴致渐无，暗自宽慰，下次一定。。。为了不再陷于伊邪那美，决定现在开始有计划地学习，顺便记录下来自己学习源码的一些经验历程。

## 写在前面

源码很大，要有计划，不然会像无头的苍蝇，陷于浩瀚源码中不能自拔。比如我目前的一些计划：要弄懂 React 整体逻辑架构（完整的`Render`渲染流程），主要注重代码的运行逻辑顺序，很多细节没必要锱铢必较，反而可能会降低学习兴趣。要弄懂` Virtual Dom Diff`算法，那么找到对应的部门源码细读就行；再比如，想搞懂`Hooks`原理，可以找到对应的文件去看在`mount`和`update`阶段分别都做了啥，以及钩子的依赖项是怎么起作用的等等。总之，要有计划，要有的放矢。

## 开发团队 Dan 神的建议

就像是想读懂一本书，有必要了解作者的背景和思想一样。想要搞懂`React`，我觉得核心开发团队的分享是最佳的第一手原味资料。

这里想先分享下作为`React.js` 的核心开发者，同时是 `Redux.js` 状态管理工具的作者 `Dan Abramov` 对于学习者的建议。（以下摘自大佬的[访谈](https://mp.weixin.qq.com/s/SBVE34dW9g4BsabmLJV9wg)）

> **主持人：很多朋友对你的个人经历感兴趣，你刚加入 Facebook 的时候是如何学习 React 的结构、概念，并慢慢开始贡献代码的呢？**
>
> Dan：整个流程最关键是要熟悉架构，一旦你对整个项目的架构有了充分的了解，接下来的所有工作就都顺理成章了。有一个对我帮助很大的点，那就是去查看人们的 `Issue`，并去解决他们。我想我个人可能已经看了数千个 Issue 了，如果你想要快速上手项目的话，我认为这是一个非常不错的学习方式。你可以点击查看处于「Open」状态的 Issue，应该差不多有 500 个，你可以从头看，也可以跳转到最后一页从最老的那个开始看。通过看 Issue 你逐渐就能理解当前有哪些问题，慢慢地去理解代码，理解项目的更新历史等等，所以这就是通过 Issue 学习的方法。

> **主持人：作为一个使用 React 的前端开发者，我们需要去阅读 React 库的源码吗？如果需要的话，有没有好的阅读代码的方式？**
>
> Dan：我觉得没有必要，这可能会是一项相当困难的工作，因为我们没有在任何其他地方提及 React 自身的架构是怎样设计的。如果你上来就开始读代码，你可能会感到非常困惑，不明白为什么这样设计。这可能也是我们将来需要改进的一点，将来到了某个时间点，我们可能会去解释这里面的实现原理。但我觉得如果你只是想玩玩的话，这个过程应该也算不上痛苦。比如我自己就很喜欢做一件事，用 `debugger` 的步进功能（`step-in`) 来一行行跑代码，看看代码会跑到哪个函数，运行代码会用到哪些不同的文件。另一件你可以做的事是使用 `Chrome Performance Tools`，你可以打开 `Chrome` 的 `Performance Tab`，点击录制，然后在你的应用中进行一些操作，之后点击停止，你会看到一张火焰图或者火焰表。这个分析结果非常有用，它就像是某种堆栈，你可以看到函数的调用顺序，看到代码中正在发生的事。它常用来测试性能，你可以看到你代码中的哪一部分运行的比较慢，但你也可以用它来做一个当前函数总览，因为上面展示了函数的名称。你可以看到状态变化会引发哪些不同的事情。你可能会发现「哦？这个函数在所有的地方都被调用了，它是做什么的？」。之后你可以点进去，看看里面到底执行了什么。沿着这种性能测试来一步步探索代码我觉得也是一个不错的学习方式，可以了解到 React 在哪部分花了时间，哪些函数是其核心之类的。

稍微跑个题，Dan 神在谈到给前端开发者的学习建议时说：总结来说，我推荐你做一些体量小，但是有深度的东西，然后从中获得收获吧。比如做一个井字棋，或者是贪吃蛇。做游戏会推动你去思考，思考程序如何去设计，思考如何去解决问题，而这一点是你平时写表单、写界面所训练不到的。我觉得这对我来说是一个很好的启发，以后的学习输出也可以以这种方式，做一些体量小、有深度的`demo`。

## React 的 Monorepo 仓库

React 采用 `Monorepo` 仓库架构，也就是一个仓库里包含了多个子仓库，`packages` 目录下可以看到很多单独的 `package`。例如比较熟悉的 `react, react-dom, react-reconciler, scheduler`等等。关于源码的概述，可以看这篇[官网源码概述](https://zh-hans.reactjs.org/docs/codebase-overview.html)。

另外，源码中经常会看见后缀 `.old.js` 或者 `.new.js` 这种文件，Dan 神是这么说的:

> 一个有趣的点，也可能是比较有争议的一个点，就是我们有很多文件都存在两个版本。如果你看源码的话，你会发现我们有 `.old.js` 或者 `.new.js` 这种文件。这些文件基本上是复制粘贴出来的，它们的内容也基本相同。我们用它们来测试一些可能会带来风险更新，对于 Facebook 网站，我们可以同时部署多个版本。我们有时候会将这些实验性的更新写到 `.new` 文件里面，然后在 Facebook 的实验环境里面进行回归测试，如果测试各项指标没有劣化的话，再将这些更新复制到 `.old` 文件中去，所以说我们的网站在任何时刻都有两个版本。在实验测试的过程中，其他人也可以去做其他部分的代码提交。这听上去可能比较奇怪，但实际用起来效果还是不错的。

## 环境配置

环境配置因人而异，可自行忽略。

平时用惯了公司的 `Mac`，这次用自己破旧的 `Windows` 给我一顿好折腾，终于找回了熟悉的感觉。

我的配置：`Win10` + `WSL (Ubuntu22.04)` + `oh-my-zsh (很爱的主题：af-magic)` + `VSCode (安装 Remote-WSL 插件直接起飞)` + `Git` + `Yarn` + `Nvm` + `Node16`。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9c92c518294c474b9aa764591269e976~tplv-k3u1fbpfcp-watermark.image?)

**VSCode 终端：**
![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d1b47d34b9bc443aa52829d6172f330f~tplv-k3u1fbpfcp-watermark.image?)

# 本地 build 流程

为了帮助开发者更快上手或者贡献代码，React 有专门的  **`Contribution Guide`**，里面就有讲怎么在本地构建代码。这里的构建流程也基本是遵循官网 [贡献流程 Contributing Guide](https://reactjs.org/docs/how-to-contribute.html)。

这里主要介绍构建源码的本地 `npm` 包后搭配 `create-react-app` (或者你自己的 react 项目) 来使用。

1. `fork` 官方仓库后，`clone` 拉取自己仓库代码。_（也可直接 clone 官方仓库代码）_

```sh
# 官方仓库代码
git clone https://github.com/facebook/react.git
```

2. `cd` 进入源码项目 `react/` 文件夹后，`yarn` 安装项目依赖。漫长的等待。。。

```sh
cd react
yarn
```

_安装截图如下，其中当前分支 `build-test` 是我从 `main` 分支上新建的分支：_
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dfd281d25e2542deb7e7167aa2895372~tplv-k3u1fbpfcp-watermark.image?)

_`yarn` 安装依赖顺利完成后会有如下提示：_

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a2bd2f10a91a488288c7db7139b09af7~tplv-k3u1fbpfcp-watermark.image?)

3. **（可选步骤，慎选）** 官方推荐使用 `yarn test` 运行单测用例来验证项目代码（慎选，又是漫长的等待，小半个小时。。。跑起来内存干满基本干不了别的）。

```sh
# 可选项
yarn test
```

4. 开始本地 `build`。这会于  `build`  文件夹中生成打包文件，这里的构建选项是选择构建基本的三个包：`react`、`react-dom`、`scheduler`。

```sh
yarn build react/index,react/jsx,react-dom/index,scheduler --type=NODE
```

- _构建顺利完成后，会输出如下信息：_

  ![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/50c0c3636f444e64b39de0a5b3920623~tplv-k3u1fbpfcp-watermark.image?)

- _可以发现打包好的代码文件放在了新增的 `build/` 文件夹下， 并且`build/node_modules/react/cjs` 目录下生成了 `CJS` 模块化规范的打包文件，如下图：_

  ![tempsnip.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5d59543d23534f1195b2cd0e3cc2993e~tplv-k3u1fbpfcp-watermark.image?)

* **\*可选打包文件类型：** 当然你也可以使用 `--type=UMD` 方式来构建 `UMD` 模块化规范的打包文件，然后通过 `script` 标签来在 `HTML` 文件中引用来使用，官方也提供了测试用的 `HTML` 文件，如下：\*

```sh
yarn build react/index,react-dom/index --type=UMD
# 然后找到打开源码中的 HTML 测试文件
fixtures/packaging/babel-standalone/dev.html
```

- _`fixtures/packaging/babel-standalone/dev.html` 代码如下，其中引用了构建后在 `build/node_modules/react/umd` 目录下生成的 `UMD` 模块化规范打包文件：_

  ![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0c649e64db344aff91dc2520ce4c91f8~tplv-k3u1fbpfcp-watermark.image?)

5. 使用 `creact-react-app` 脚手架来快速创建自己的 React 项目：

```sh
# 返回 react 源码同级目录，初始化自己的应用
cd ..
# 给自己应用命名 my-app
npx create-react-app my-app
```

5. 为了在自己应用中能够使用我们上一步在本地构建好的源码 `npm` 包，我们需要通过 `yarn link` 修改依赖项的软连接指向来让其生效。

```sh
# 进入源码项目 react 路径
cd react
cd build/node_modules/react
yarn link
cd build/node_modules/react-dom
yarn link

# 进入我们自己应用 my-app 路径
cd my-app
yarn link react react-dom
# 编译启动
npm start
```

6. 最后，我们修改源码增加打印输出，然后重新 `npm start` 启动我们项目来验证下效果。

   _修改 `react/build/node_modules/react/cjs/react.development.js` ：_

   ![tempsnip-1.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/31cdefb0336949e285e5425603c7fa10~tplv-k3u1fbpfcp-watermark.image?)

   修改 `react/build/node_modules/react-dom/cjs/react-dom.development.js` ：

   ![tempsnip-3.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c8711248c32a4ecea15c4342db0998df~tplv-k3u1fbpfcp-watermark.image?)

   👏 可以看到，刚刚修改的源码已经生效，现在我们可以开始随意修改调试源码了。

   ![tempsnip-console.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bacf0a1540974588992c05986c38c7e8~tplv-k3u1fbpfcp-watermark.image?)

# 补充：build 报错修复

在 `yarn` 安装依赖过程中可能遇到报错如下, 提示找不到 `autoreconf`:

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/598dc8d7138a4800817412be5fd2d67f~tplv-k3u1fbpfcp-watermark.image?)

**解决：** 安装报错提示缺少的依赖，然后再次 `yarn`：

```sh
sudo apt-get install autoconf automake libtool
yarn
```

# 相关资料

官方社区里也有很多源码分析相关的博客文章。

1. 强烈推荐 Dan 神的个人博客，网址名称 overreacted 也很有意思: https://overreacted.io
2. Behind the Scenes: Improving the Repository Infrastructure:<https://reactjs.org/blog/2017/12/15/improving-the-repository-infrastructure.html>
3. Sneak Peek: Beyond React 16: <https://reactjs.org/blog/2018/03/01/sneak-peek-beyond-react-16.html>
4. 关于 Monorepo: https://mp.weixin.qq.com/s/U8_30S9B0S_SU3jdgUxFGQ
5. （自己搭建一个 React 框架）Build your own React: https://pomb.us/build-your-own-react/
