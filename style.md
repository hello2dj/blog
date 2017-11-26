1. 对于一个给定的元素E，当有E有一个属性P的值发生改变时， E的子孙元素F需要重新计算他的样式仅限于一下两个条件同时存在一：P有一个集合descendant invalidation set S 二：F上面有一个属性值是集合S的成员，属性P可以使类名，id值，或者其他属性值
2. 当invalidation set是空的时候，意味着只需要重新下计算属性改变的那个元素的样式
3. 有一个wholeSubtreeInvalid的标志与每一个invalidation set相关联，当他的值是true的时候，意味着他的所有的子孙都需要重新计算样式
4. 有一个treeBoundaryCrossing的标志与每一个invalidation set相关联，当他的值是true的时候，意味着invalidation set 使得shadow trees的元素也发生了改变
5. 有一个insertionPointCrossing的标志与每一个invalidation set相关联，当他的值是true的时候，意味着the invalidation set invalidates elements distributed under content element descendants.

//其实就是从右到左依次将属性p添加到下一个属性P的invalidation set中，只有一些特例而已
Constructing Invalidation Sets
Invalidation sets are constructed and aggregated by iterating through all CSS selectors. For each selector we go through a two-step process where we first extract properties from the simple selectors of the rightmost compound selector which we then add as properties to the invalidation sets for properties found by looking at simple selectors in the other compound selectors.

The simple selectors from rightmost compound selector are the interesting properties to be added to the invalidation sets since those selectors are the ones that will match the element that will get the declarations from the style rule. For instance, the rule:

.a .b { color:green }

will contribute to the computed style of elements with class b while elements with class a will just take part in the selector matching.

There are some simple selectors which are currently skipped and not added to the invalidation sets. One example is negated selectors. It is not impossible to implement, but we currently do not support negated members of invalidation sets.

The algorithm per selector can be described as:

算法如下：
  1. 对于简单的选择器按照从右到左的方式
    1. 把P添加到一个临时的set S中（对于一个css rule来说这个S是一个局部全局的）
    2. 如果P没有 descendant invalidation set， 给P创建一个空的invalidation set
  2. 对于其他组合选择器
    1. 对于每个简单选择器映射到P的invalidation set
      1. 如果没有 invalidation set ，创建一个
      2. 
For each simple selector in the rightmost compound selector that maps to an invalidation set property P:
Add P to a temporary set S.
If there is no descendant invalidation set for P, create an empty invalidation set for P.
For all other compound selectors
For each simple selector that maps to an invalidation set property P
If there is no invalidation set for P, create an invalidation set for P.
If S is empty, or if the combinator right of the compound selector is an adjacent combinator, set the wholeSubtreeInvalid flag on the invalidation set for P.
Otherwise add all members of S to the set for P.
If there is a ::content pseudo element anywhere right of the simple selector for P, or the simple selector for P is in a :host or :host-context pseudo class, set the insertionPointCrossing flag on the set for P.
If there is a ::shadow or /deep/ combinator anywhere right of the simple selector for P, or the simple selector for P is in a :host or :host-context pseudo class, set the treeBoundaryCrossing flag on the set for P.

There is another complexity to P extraction from the rightmost compound selector when you have pseudo classes which takes a compound selector list like :-webkit-any(). These selector lists are disjunctions in the sense that only one compound selector needs to match in order to match the whole pseudo. Consequently, -webkit-any(*, .a) is a universal selector when present in the rightmost compound as far as the invalidation sets are concerned. Yet, when present in other compound selectors, Ps from all selector list compounds need to have their invalidation sets constructed. See the Examples section.
Notation
In the examples and text below, we use the following notation:

a - element name
.a - class name
#a - id
[a] - attribute name
* - wholeSubtreeInvalid
! - treeBoundaryCrossing
!! - insertionPointCrossing

P { P0, P1, .. , PN } - invalidation set for P

Examples
Selectors:

.a { }

Invalidation sets:

.a { }

Selectors:

.a .b { }
.c { }

Invalidation sets:

.a { .b }
.b { }
.c { }

Selectors (:not(.b) does not map to a P, so there are no Ps in the rightmost compound which causes the wholeSubtreeInvalid to be set for the set for “.a”):

#x * { }
#x .a { }
.a :not(.b) { }

Invalidation sets:

.a { * }
#x { *, .a }

Selectors (treeBoundaryCrossing and insertionPointCrossing):

:host-context(.a) span { }
:host(.b) input[disabled] { }
[type] ::content > .c { }

Invalidation sets:

.a { span, !, !! }
.b { input, [disabled], ! }
.c { }
[type] { .c, !! }

Selectors (disjoint selector lists):

.a :-webkit-any(:hover, .b) { }
.c :-webkit-any(.d, .e) { }
:-webkit-any(.f, .g) { }
:-webkit-any(.h, *) #i { }

Invalidation sets:

.a { * }
.b { }
.c { .d, .e }
.d { }
.e { }
.h { #i }

Selectors (negated simple selectors mapping to properties in non-rightmost compound selectors):

:not(.a):not(.b) > span#id { }

Invalidation sets:

.a { span, #id }
.b { span, #id }

Selectors (adjacent selectors causing wholeSubtreeInvalid):
	
.a + .b #id { }
.c ~ span { }
.d ~ [type] div { }

Invalidation sets:

.a { * }
.b { #id }
#id { }
.c { * }
.d { * }
[type] { div }

Conclusion
Although Descendant Invalidation Sets will give you false positives when marking elements for style recalculation, it has proven to reduce the number of elements affected during style recalculation drastically, and hence reduced occurrences and severity of janks.
