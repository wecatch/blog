title: 为什么选择 emberjs 开发 dashboard 和 CMS 类系统 
date: 2016-06-04 18:35:35
tags:
- emberjs
categories:
- 技术
---


[原文链接](http://sanyuesha.com/2016/06/04/why-emberjs-is-best-for-cms/)

## emberjs 是什么

在开始之前，先看一组 github 数据

**ember.js**

- star 16291
- contributors 586
- releases  181
- issues 167 open, 4397 closed

https://github.com/emberjs/ember.js

**angular.js**

- star 49643
- contributors 1476
- releases 169
- issues 817 open, 6992 closed

https://github.com/angular/angular.js

**vue.js**

- star 19883
- contributors 69
- releases 137
- issues 27 open, 2252 closed

https://github.com/vuejs/vue


**react.js**

- star 43026
- contributors 713
- releases 46
- issues 480 open, 2843 closed

https://github.com/facebook/react


**riot.js**

- star 9565
- contributors 138
- releases 137
- issues 96 open, 1089 closed


https://github.com/riot/riot

> 注: 统计时间为本文编写之时

emberjs 和其他同类框架一样，至少在 github 的 表现足以证明其流行度是不低的，但各个框架的诞生的场景以及解决的问题又不尽相同。

emberjs 是基于 jquery 的一个全栈式前端框架，它提供了从路由层-> view 层->数据和网络交互层的全部解决方案，只用 emberjs 就足以解决前端单页应用遇到的所有复杂问题，而且兼具有不俗的开发效率和开发质量。


#### 全栈式解决方案

emberjs 是一个大而全的框架，它几乎囊括了单页应用开发所需要的方方面面: component，model，service，route，temlate
，ember-data。借助 emberjs 几乎可以完成任何规模的复杂应用，而不用担心它没有提供足够丰富的特性或者需要求助其他第三方库(不一定可以很好的和框架本身集成)。

<!-- more -->

#### 基于 jquery 

emberjs 是完全基于 jquery 的，意味着任何基于 jquery 的第三方库都可以和 ember app 集成，而这些年构建于  jquery 之上的框架和插件不计其数，在没有必要重复造轮子的地方，完全可以用丰富的第三方轮子替代，比如 bootstrap，semantic-ui 这种大而全的 UI framework，jQuery-File-Upload 这种可以兼容各种主流浏览器的文件上传组件。利用这一优势，大量现成的 jquery 插件能很快集在 ember app 中，借助 ember component 又可以很轻松达到复用的目的。

#### 良好的兼容性

最低能够兼容到 IE 8，新版本也在逐步淘汰低版本IE的兼容。

#### 约定优于配置

开发、管理和维护复杂的单页应用，是一件非常考验组织架构的事情，组件的复用、网络的交互、全局状态的管理、复杂的表单数据、路由等都需要经过精心设计和严格的代码规范才能最大限度的保证在多人协同开发之下，代码仍具有良好的维护成本和高效高质的产出。emberjs 秉承约定由于配置的理念，已经在框架层面把应用的最基本结构确定了，开发者只需要顺势而为，即可保证代码的可维护性和可协作性。


#### 数据的双向绑定

不同于在 react 中，数据只能单向流动，emberjs 中的数据默认是双向的，这意味着数据的变更完全交由框架处理，虽然双向数据绑定在复杂应用中易于导致数据的状态变更不易跟踪和维护，但是对于频繁且复杂的表单处理，双向绑定可以极大的提高开发效率，通过一定的约束和规范，让双向绑定在可控的范围内发生，并不会对维护造成太大的影响。


#### 传统的字符串模板

对于习惯了传统模板的很多后端程序员来说，字符串模板非常易于接受和理解，几乎没有学习成本，而且能很好的反应 html 结构和视图，这点相对于 react 的 jsx 语法就有很大的不同。这点也足以构成像我这样的后端程序员偏爱 emberjs 的一个理由。


#### 完备的构建工具和构建流程


emberjs 社区提供了完备的构建工具和构建流程来保证 ember app 的开发和发布效率，基上能完成在多种环境之下的一键开发和部署，非常方便。


#### 渐进式的向下兼容

前端的发展日新月异，新技术层出不穷，要想一直惠于新技术和新方案的发展，需要不断的升级优化。这也带来了不小的升级维护开销。如果一个框架完全不考虑遗留系统，升级代价巨大，对于人力紧缺，时间宝贵的小团队来说，这样的代价负担不起。emberjs 很好的做到了这一点，基本每个版本都可以做出很平滑的过渡，而不是大规模的修改。在这一点上，angular 就显得非常激进。


## emberjs 究竟适合什么样的场景

- 全栈式解决方案
- 基于 jquery 
- 约定优于配置
- 数据的双向绑定
- 传统的字符串模板
- 完备的构建工具和构建流程

基于这些特点，emberjs 非诚适合于

- 需要一体化解决方案(懒)
- 不会特别在乎体积，不会特别在乎性能(用的人不是很多，没有时间优化)
- 需要一个大而全的 css framework，类似bootstrap，semantic-ui，foundation(还是懒)
- 表单处理复杂(频繁创建和修改，查询)
- 尽量利用第三方高质量的库来解决现成问题(缺乏足够的时间和精力造轮子)
- 非专业前端程序员开发和维护(后端程序员开发和发布)
- 不能花太多时间进行组织架构和代码规范(业务变更频繁，多人交叉开发，缺乏足够的经验和时间进行架构设计，缺前端)

具有以上特点的各种复杂的 dashboard 和 内容管理应用。这类应用基本具备性能要求不高，但是需求变更频繁，表单处理复杂，由于频繁变更后端逻辑和数据结构，页面的组织结构和交互需要频繁修改。高频变更之下，要保证代码具有很强的一致性和可维护性，ember 约定优于配置的理念恰如其分得契合这样的需求。

## emberjs 的问题

1.体积庞大

emberjs 的体积非常庞大


| Framework              | Version    | Minified Size (gzip) |
|------------------------|------------|----------------------|
| Ember                                          | 2.5.0      | 117.26kb             |
| Polymer + Web Components Polyfill Lite         | 1.4.0      | 54.48kb              |
| Angular                                        | 1.5.0      | 53.17kb              |
| React                                          | 15.0.2     | 43.62kb              |
| Web Components Polyfill                        | 0.7.22     | 33.66kb              |
| Vue                                            | 1.0.21     | 25.98kb              |
| Riot                                           | 2.4.1      | 9.23kb               |


单从体积上看，emberjs 显得过于雍正，但是在很多场景之下，体积根本不是需要考虑的因素。


2.性能

由于 emberjs 提供了单页应用开发所需要的几乎全部解决方案，相比于像 vuejs、reactjs、riot等这些专注于 UI 层的框架的的性能就有了一定的差距，再加上字符串模板天生特性和 emberjs 实现双向绑定的机制，性能不是很好是 emberjs 的一个问题，但是并不代表 emberjs 没有优化的空间。[discourse](http://www.discourse.org/) 就是很好的案例。点击查看 discourse 团队对 emberjs 的优化 [eviltrout](https://eviltrout.com/)。

3.灵活性差

大而全框架天生的缺陷

4.学习曲线陡峭


emberjs 特性众多，开发环境相对复杂，需要较长的时间入门和实践，尤其是对刚接触全栈式前端框架的初学者来说，要很好掌握component 交互、单双数据绑定使用场景、插件开发等需要花一定时间学习。

emberjs 的开发一旦掌握，开发效率会出奇得快。


## 选择合适的才是最好的

我们的团队在生产环境中使用了 reactjs、emberjs、vuejs、riot，还有部分 angular 遗留项目。

angular 是最初使用 js framework 时候的一个尝试，生产环境没有做过大规模得实践，现在基本放弃。

riot 是个优秀框架，体积小，性能优越，灵活性极强，也正是这点，在代码量达到一定规模之后，如果架构设计跟不上代码量和复杂性的增长，可维护性会越来越差。riot 的轮子也很少。

react 的出现带来了前端跨越式地发展，也带来了全新的思维方式来构建前端 UI 层，几乎是一个全新生态，正因为如此， react 使用的所有轮子都需要重新打造才能完全发挥 react 带来的优势。还有一点是，JSX 的语法也完全不符合代码即视图的设计理念，需要时间适应。

vue 同样是个十分优秀的框架，接口表现力强，学习成本低，能够很快入门，同样 vue 保留了 html 结构即视图的模板概念，使用起来非常顺手，结合 vue-router 和 vuex，一样可以构建非常复杂的应用。vue 的生态也缺乏足够多的轮子。

对于面向大规模用户、性能要求卓越、灵活性强的、轮子不得不造、多人协作的场景，react 和 vue 是更合适的选择。但是面对类似 dashboard 和 CMS 这种表单处理众多、需求复杂的、变更频繁的业务场景，emberjs 可能是更好的选择。


- 国内第一个 emberjs 讨论区 [emberjs-china](http://emberjs-china.org/)
- 基于 semant-ui 的 ember addon [ember-semantic-ui](https://github.com/wecatch/ember-semantic-ui)
- 比 ember-data 简单很多的 ORM [ember-easy-orm](https://github.com/wecatch/ember-easy-orm)


