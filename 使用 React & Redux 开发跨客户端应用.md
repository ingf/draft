此文深受 @xufei 的 [一系列博客](https://github.com/xufei/blog/issues) 影响，亦有很多引用，非常感谢。

> 客户端（Client）或称为用户端，是指与服务器相对应，为客户提供本地服务的程序。一般安装在普通的客户机上，需要与服务端互相配合运行。因特网发展以后，较常用的用户端包括了如万维网使用的网页浏览器，收寄电子邮件时的电子邮件客户端，以及即时通讯的客户端软件等。

在这里，我们说的客户端应用包含桌面浏览器和移动设备浏览器（或者其他 Web 容器）中运行的 Web App，iOS 和 Android 上的原生应用，Mac 和 tvOS 上的原生应用等等。

对于一家公司的客户端产品而言，或多或少会运行在上述的几个平台之上。而这些客户端产品中，基础的业务逻辑是相同的，稳定的。那么你肯定猜出来了，我们希望能够在多个客户端产品中，共用这部分基础业务逻辑，然后上层根据不同的平台和产品设计实
现相应的 UI 展示。

实现这个目标是比较难的，先看看 Web 前端普遍存在的情况：一个产品，不管他在初期还是成熟阶段，产品经理总会花点时间出来，整个改版，这个改动主要是在 UI 层面，当然也会涉及一部分的产品逻辑调整。这时候前端工程师就不淡定了，我是改改 UI 呢还是重构呢还是重构呢😂

此时，工程师大多会思考，如果把业务逻辑和 UI 层分开多好呀。

> UI 层（View） ---- 业务逻辑层（Controller）

如果把 Model 层和业务逻辑放在一起，那么就是酱紫

> UI 层（View） ---- 业务逻辑层（Controller） ---- 数据层（Model）

对，这就是我们经常说的 MVC，而他最终很有可能变成了这个样子。

![mvc](https://cloud.githubusercontent.com/assets/3368034/13290550/0833292a-db4f-11e5-8f6e-e59386f33305.png)

这张图片是 Facebook 软件工程师 Jing Chen 在 [F8](https://www.youtube.com/watch?v=nYkdrAPrdcw)上面提出来的，实际情况不是所有项目都有这么严重，但是比这严重的项目肯定有很多。这并没有从根本上将 UI 层和业务逻辑分开，还是耦合在一起，怎么样才能真正将 UI 层和业务逻辑层分开来呢，我们再来看看这张图。

<img width="610" alt="redux" src="https://cloud.githubusercontent.com/assets/3368034/13290565/13916caa-db4f-11e5-9189-78cff60ee311.png">

上图的 React Component 就是 UI 层，Action，Store，Reducer 就相当于业务逻辑层，其中 store 就是一个大 Model。
UI 层和逻辑层的关系：UI 层的数据来源于逻辑层的 Store，必要的时候，UI 层通过调用 Action 修改 Store。采用这种方式，我们能比较从容的面对产品改版，因为修改主要集中在 UI 层，以前的逻辑层稍作修改就好。目前 UI 层是 React，可以实现在浏览器上面的需求，如果我们再将 UI 层稍微修改，换成 [React-Native](https://facebook.github.io/react-native/)、
[React-tvOS](https://github.com/jordanbyron/react-native/tree/tvOS)、
[React-Desktop](https://github.com/ptmt/react-native-desktop)，甚至还可以换成 Angular 和 Vue，那么就实现了我们初期的目标。在底层，有稳健的业务逻辑实现，在 UI 层，可以使用合适的熟悉的框架，甚至可以更换 UI 层框架。

![screenshot 2016-04-17 20 17 19-min](https://cloud.githubusercontent.com/assets/3368034/14586925/74613fce-04da-11e6-9e8b-7266392236d4.png)

当然，实现 UI 层和逻辑层的分离可以很多方案，我们这里是采用 Redux 作为逻辑层的实现。先扔出结论，接下来我们详细讨论一下这种分层、组件化的开发方式，先看一下 Web 前端组件化的一些理念。
### 前端组件的复用

在大型软件中，组件化是一种共识，它一方面提高了开发效率，另一方面降低了维护成本。但是在 Web 前端，还没有很通用的组件模式，因为缺少一个大家都能够认同的实现方式。所谓组件化，核心意义莫过于提取真正有复用价值的东西。那怎样的东西有复用价值呢
- UI 控件
- 基础逻辑功能
- 公共样式
- 稳定的业务逻辑

以上几点在 @xufei 的[2015前端组件化框架之路](https://github.com/xufei/blog/issues/19)中已经说的很清楚了，不再多言。除此以外，还有大量的业务界面，这部分基本不具备复用性，但是大部分还是会把他们组件化，也就是把大块的业务界面，拆分成若干小块组件，然后进行组装。这并不是为了复用，纯粹是为了使得整个工程易于管理，易于维护，组件化的最重要作用就是提升开发和维护的效率。
### 组件化的主要目标

很多人会把复用作为组件化的第一需求，但实际上，在 UI 层，复用的价值远远比不上分治，将代码从 UI 层和逻辑层分开来，就是一种分治。

分治带来的是可管理性，相比一大团 HTML 和 JavaScript 的混杂，组件化之后，整个应用成为了一个很清晰的树，一眼就能看清包含关系，也能够很容易理清数据的传递方向。而且，整个应用可以从叶子节点，逐步向上测试，哪一级出了问题，可以很容易发现。

但是复用就很麻烦了，因为组件的内部实现与外部接口都很难取舍。很可能我们在设计之初，都是把组件设想成一个单一的东西，然后在实际项目中，发现最后都面目全非了。

所以，复用的工程成本很高，在使用的时候需要权衡，除了最常用了基础控件，其他的不要刻意追求。

这个时候，思路就比较清晰了，基于组件化思想，应用分成 UI 层和逻辑层，逻辑相对比较稳定，采用 Redux 方式实现，UI 层也采用组件化方式，不过这种组件化并不是为了复用，而是从工程角度来说，为了便于理解、管理和维护。我做了一个 demo- [captain](https://github.com/ingf/captain)，目前仅实现 Web 和 iOS 平台。

在深入之前，先看看 captain 的目录结构：

```
.
├── README.md
├── android            // Android 平台 UI 实现
├── app                // 业务逻辑
├── index.android.js   // Android 入口文件
├── index.html         // 
├── index.ios.js       // iOS 入口文件
├── index.web.js       // Web 入口文件
├── ios                // iOS 平台 UI 实现
├── server.js          // 
├── test               // test
├── web                // Web 浏览器 UI 实现
└── webpack.config.js
```
### 业务逻辑

> 业务逻辑指的是所处领域中的业务数据、规则、流程的集合。即使抛开所有展示层，这一层也是应当要能够运作起来的。

我们采用 Redux 来开发业务逻辑，当然这个不是绝对的。如果对于 Redux 还不太了解，请移步 [这里](http://redux.js.org/)。Redux 总结起来就一句话：

> 将 **动作(Action)** 通过 **状态转换函数(Reducer)**，set 到一个统一的地方(**Store**)，然后 UI 从 Store 中获取数据。

逻辑层应该是与 UI 层无关的，要能偶独立运作起来，能够独立测试。

在 captain 的 test 里面，实现了一套简单的逻辑测试，启动后可以在[这个网页](http://localhost:9527/test)的控制台里面看到效果，下面有截图。

```
store.dispatch(actions.addTodo('Learn about actions'))    // 添加一个 todo
store.dispatch(actions.addTodo('Learn about reducers'))   // 添加一个 todo
store.dispatch(actions.addTodo('Learn about store'))      // 添加一个 todo
store.dispatch(actions.completeTodo(0))                   // 完成第零个 todo
store.dispatch(actions.completeTodo(1))                   // 完成第一个 todo
store.dispatch(actions.clearCompleted())                  // 清除已经完成的 todo
```

<img width="755" alt="actions" src="https://cloud.githubusercontent.com/assets/3368034/13290582/267cb9fa-db4f-11e5-89b3-227ca6b42c8f.png">

简直没有比这再清晰的了，你甚至都不用阅读我的源码，只需要看一下这个 Action 列表（每个 Action 日志中会打印出初始状态、执行动作和执行后的状态），就知道业务逻辑是怎样执行的，再也不会出现 MVC 中一个 Model 更新了以后不知道哪些 View 会随之更新的情况了。这正是 Redux 的一个很大的优点 ---- 可预测性。正因为可预测性，你清楚的知道什么发生了改变（Action），改变之后的数据是什么样的（Store/State），以及发生了哪些改变（Action 记录）。

实际在开发 Redux 应用的过程中是比较折腾的，尤其是在前期，生态圈不够完善，缺乏大型应用的成熟案例等等。但是当我们慢慢摸着石头过到河中间，所以痛点都找到化解的方式之后，好像突然架起了一座桥，开发效率 biu 的一下就上去了。其实我认为 Redux 最能提升的不仅仅是开发效率，更是维护效率。若干年以后（如果产品还活着），你看到上图中打印出来的信息，还是能清楚的知道这个业务过程是怎样的。关于维护效率，我想多说一句，几乎每个大公司都有一个“运行时间长，维护的工程师换了一批又一批”的项目。Amazon 曾经有个工程师描述维护这种项目的感觉：“climb the shit mountain”，所以我猜想你能感受得到参与一个好维护的项目是怎样一种体验。
### UI 层

UI 层与普通的 React 项目和 React-Native 项目一致，可参考：https://github.com/ingf/ingf.github.io/issues/2
