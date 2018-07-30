---
title: mpa小札
date: 2018-07-29 16:56:47
tags: [pwa, mpa, 技术, 杂文]
---

Spa 使得前端项目真正得以工程化独立化，脱离后端约束。

而 Single-spa 则通过配置的方式使得多框架多单页应用成为现实，一个项目拆分成多个独立的子项目，按需加载。

基于 single-spa 的 Mooa 则通过 容器的理念以及主-从设计，使得逻辑上划分代码区域的子应用在物理上也完全区分开来。多个应用独立开发，独立部署，互不影响。但展示给用户的，却是一个完整的应用。

<!--more-->

![image](http://b289.photo.store.qq.com/psb?/V13DGKvw03VhSb/zeqjAmwbeH9.UkEpnfMhmHPXpeX6oMpr4LPBnQcLwAI!/b/dCEBAAAAAAAA&bo=0gOTBdIDkwUDIAU!&=viewer_311)

虽然 Mooa 是为 angularJs 所开发，有着语言的限制，而且目前 gitHub 也并不活跃，但是管中窥豹可见一斑，spa 虽然足够优秀，打包体积过大也可以通过懒加载等方式优化，然而当一个系统很大很复杂的时候，服务端可以通过微服务来保证项目的稳定性和持续集成，同时针对不同的情况可以选择不同的语言，而前端，则很难如此。传统的更不必说，spa 也会限制在某一种框架，然而，一个项目变得庞大起来，整套代码都会越发臃肿，势必要从底层开始优化，通过约定和规范来尽量保证代码的可读性和可维护性。这大概就是当前开发的一个痛点。而 Mooa 就是为了解决这个痛点，它的技术并不成熟，所谓的容器也只是对 iframe 的包装。但是，通过这种包装，的确可以支持在一个 web 项目里容纳多套单页应用。或者，可以称之为“前端开发微服务化”。当下流行的 React,Vue 都是组件化的思想，而组件本身就是比较接近微服务的思想。

![image](http://a2.qpic.cn/psb?/V13DGKvw03VhSb/j2T*3QZr1jqqFP68vzeCqVtzm7R.*LSvUcixwtAOA9M!/b/dCEBAAAAAAAA&ek=1&kp=1&pt=0&bo=iAcYBIgHGAQRECc!&tl=1&vuin=2295161275&tm=1532854800&sce=50-1-1&rf=viewer_311)

![image](http://a3.qpic.cn/psb?/V13DGKvw03VhSb/DvCTogNLsxeDYfQaLgzjHejlhnFK1OT6TmjdBiQZV6s!/b/dCIBAAAAAAAA&ek=1&kp=1&pt=0&bo=OAR.BjgEfgYRECc!&tl=1&vuin=2295161275&tm=1532854800&sce=50-1-1&rf=viewer_311)

我想，未来某一天，这种思想和技术都趋近成熟，三五人直接开发小型 web 应用，且可以不依赖其他环境直接部署，或直接作为一个独立网站应用，或作为某个网站的一部分。

PWA 是未来 WebApp 展现给用户的方式，从视觉上打破了浏览器的限制，当下的快应用即属于一种变体。

而前端独立开发独立部署自由组合，则会是 WebApp 开发的一种方式。
