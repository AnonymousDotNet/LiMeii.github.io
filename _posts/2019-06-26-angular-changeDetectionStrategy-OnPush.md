---
title: Angular Change Detection:OnPush
tags: Angular
layout: post
---

在[Angular Change Detection:变化检测机制](https://limeii.github.io/2019/06/angular-changedetection/)这篇文章介绍了angular的变化检测机制，也提到了页面操作（click，submit...）、XHR、Timers（setTimeout()，setInterval()）这些异步事件都会触发整个angular application的变化检测。这是angular默认的变化检测机制（**ChangeDetectionStrategy.Default**）。


但是在实际应用里，并不是每个异步操作需要变化检测，某些组件也可以完全不用做变化检测，过多的变化检测实际上也影响了整个应用的性能。angular除了默认的变化检测机制，也提供了**ChangeDetectionStrategy.OnPush**，OnPush可以跳过某个component或者某个父组件以及它下面所有子组件的变化检测。


我们来看下具体是怎么用的：


**Immutables and Observables**


- Input reference of the component changes

- DOM Event within a component has been dispatched (ex. click)

- Emission of an observable event subscribed with Async pipe

- change detection is manually run ：ChangeDetectorRef.detectChanges()/ ChangeDetectorRef.markForCheck()/ApplicationRef.tick()


**未完待续**