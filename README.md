# risc-v-roam

## 是什么
记录我在 [Open-Source OS Training Comp 2022](https://github.com/LearningOS/rust-based-os-comp2022) 的所学

## 学习记录

2022/6/29 
- 18:35 The Genesis 学习记录仓库 commit
- Some rustlings :smile: 做了点题找回手感

2022/6/30
- Some rustlings 

2022/7/1
&nbsp;&nbsp;顶着大太阳军训真是让人无语，还偏偏就我们排全程大太阳:sunny:，真亏我能挺过来 (明明说好五分钟军姿，怎么每次都是十分钟哇:sob:)
- Some rustlings 进度推得有点慢啊……
- match 模式匹配会尽可能地消费，所以在一定情况下需要使用 `ref` 关键字来取引用
> Bind by reference during pattern matching.

> ref annotates pattern bindings to make them borrow rather than move. It is not a part of the pattern as far as matching is concerned: it does not affect whether a value is matched, only how it is matched.

> By default, match statements consume all they can, which can sometimes be a problem, when you don’t really need the value to be moved and owned


