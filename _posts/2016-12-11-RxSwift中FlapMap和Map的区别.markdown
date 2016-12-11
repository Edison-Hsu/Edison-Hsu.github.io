---
layout: post
title: RxSwift中FlatMap和Map的区别
date: 2016-12-11T17:23:00+08:00
categories: 总结 RxSwift
published: true
---

# 从头说起
要深入了解这两个的区别以及一些使用场景，就必须知道几个基本的概念，一个是 Swift 中的 FlatMap 和 Map， 另一个就是迭代器的概念。

## Swift中的FlatMap和Map
首先来看看 Swift 中的 FlatMap 和 Map 这两个语言原生的概念，来看下面代码:

```
let array = [1,2,3,4,5]

let maped = array.map { "No." + String($0) }
print(maped) // ["No.1", "No.2", "No.3", "No.4", "No.5"]

let flapMaped = array.flatMap { "No." + String($0) }
print(flapMaped) // ["No.1", "No.2", "No.3", "No.4", "No.5"]

let arrayNested = [[1,2,3,4,5],[6,7]]

let maped2 = arrayNested.map { $0 }
print(maped2) // [[1, 2, 3, 4, 5], [6, 7]]

let flaped2 = arrayNested.flatMap { $0 }
print(flaped2) // [1, 2, 3, 4, 5, 6, 7]
```

在这个例子中，区别很明显，就是当数组是嵌套数组的时候，这个时候 flatMap 会将数组中的元素压平，这就是最简单的例子。

## 迭代器
首先我们知道 Swift 中，数组可以使用 flatMap 和 map 并且是一个迭代器。追溯源码可以发现一个 SequenceType 类，这是 Swift 迭代器的基类，并实现了 flatMap 和 map 的方法。
然后 RxSwift 中，最基础的概念就是 Observable。翻看文档会发现：

> 每个 Observable 序列只是一个序列。一个 Observable相较 Swift 的SequenceType 的核心优势是它还能异步的接收元素。这是 RxSwift 的核心， 这里的文档是关于我们在那个想法上的展开。

所以，简单来讲，Observable 也是一个迭代器，只是说这个是异步的。
我们用迭代器这个概念，联结了 Observable 和 SequenceType。
这就是理解这两个函数最核心的概念了。

## 结合起来看
最后来看一个 RxSwift 的例子：

```
example("flatMap and flatMapLatest") {
    let disposeBag = DisposeBag()
    
    struct Player {
        var score: Variable<Int>
    }
    
    let 👦🏻 = Player(score: Variable(80))
    
    let player = Variable(👦🏻)
    
    player.asObservable()
        .flatMap { $0.score.asObservable() } 
        .subscribe(onNext: { print($0) })
        .addDisposableTo(disposeBag)
    
    👦🏻.score.value = 85
    
    👦🏻.score.value = 95 
}
```

Varable 是 RxCocoa 中的一个概念，在这里，我们只需要简单的把他看做 Observable 就行，具体的解释可以参考我翻译的 RxSwift 的[文档](https://github.com/jhw-dev/RxSwift-CN)

上面的例子就很清楚了，男孩.score 是一个 Observable，player 本身也是一个 Observable，所以在这个例子中需要观察男孩的成绩就需要用到 flatMap。
