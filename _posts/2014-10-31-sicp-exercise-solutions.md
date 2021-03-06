---
layout:   post
title:    SICP习题和解
date:     2014-10-31 20:57
summary:  最近在啃《计算机程序的构造和解释》，我会把习题陆续更新上来
---

最近在啃《计算机程序的构造和解释》，以下是我对习题的理解，陆续更新中  

## 1.1 程序设计的基本元素

1.1 过于简单,略过

1.2 将表达式转换为前缀表达式

{% highlight scheme %}
(/  ( + 5 4 (- 2 ( - 2 (- 3 (+ 6 (/ 4 5))))))
    ( * 3 (- 6 2) (- 2 7))
{% endhighlight %}

1.3 求三个数中，较大的两个数之和（我的思路:找出最小的，然后计算剩下两个的和，当然你也可以先找出最大的，然后再找第二大的）:

{% highlight scheme %}
;sum-of-top-2.lisp

(define (>= a b)
    (not (< a b))
     )

(define (sum-of-top2 a b c)
    (cond   ((and (>= a c) (>= b c)) (+ a b))
                ((and (>= c a) (>= b a)) (+ b c))
                            (else (+ a c))
    ))

(define (a-plus-abs-b a b)
    ( (if (> b 0) + -) a b))
{% endhighlight %}

1.5 理解的关键在于 (define (p) (p)) 是一个无限死循环，只要分析正则序和应用序是怎么展开的就简单了。用正则序展开如下:

{% highlight scheme %}
; 原表达式
(test 0 (p) )

; 1 扫描到test操作 ==> 展开
(( if (= 0 0)
       0
       (p) ))

; 2 扫描到 if 操作，而且(= 0 0) 已经完全展开，值为真 ==> 求值
0 
{% endhighlight %}

正则序会返回结果0，我们再看看应用序：

{% highlight scheme %}
; 原表达式
(test 0 (p))

; 1 扫描到test的时候，并不会展开，因为应用序优先求操作对象的值
; 两个操作数，0的值不用算，p的值需要计算:
(test 0 ((p)) )

; 2 继续求p
(test 0 (((P))) )

; 3 ... 无限循环
(test 0 (((((.... p ))))....  )
{% endhighlight %}

所以，这题的答案是:正则序返回 0 ， 应用序会无限循环
