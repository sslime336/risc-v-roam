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
- Some rustlings 进度推得有点慢……还是有收获的，有些情况在我自己使用 Rust 的时候并没有遇到或考虑过
- match 模式匹配会尽可能地消费，所以在一定情况下需要使用 `ref` 关键字来取引用
> Bind by reference during pattern matching.

> ref annotates pattern bindings to make them borrow rather than move. It is not a part of the pattern as far as matching is concerned: it does not affect whether a value is matched, only how it is matched.

> By default, match statements consume all they can, which can sometimes be a problem, when you don’t really need the value to be moved and owned

2022/7/2  

&nbsp;&nbsp;今天军训到晚上……只做了一道题。看了看别人的解答突然觉得自己像个傻子:cry:
- 1 rustlings problem

### Rustling Progress Currently 
*(temporary echo)*
- [ ] advanced_errors/
- [ ] clippy/
- [ ] collections/
- [ ] conversions/
- [x] enums/
- [x] error_handling/
- [x] functions/
- [x] generics/
- [x] if/
- [x] intro/
- [ ] macros/
- [x] modules/
- [x] move_semantics/
- [x] option/
- [x] primitive_types/
- [x] quiz1.rs
- [x] quiz2.rs
- [x] quiz3.rs
- [ ] quiz4.rs
- [ ] standard_library_types ← `pivot`
- [x] strings/
- [x] structs/
- [x] tests/
- [ ] threads/
- [x] traits/
- [x] variables/