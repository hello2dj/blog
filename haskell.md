### 部分函数和全函数
  * #### 全函数对所有的合法输入都有正确的返回
  * #### 部分函数对合法输入的子集能够正确处理，其他的则会异常
    ```
    -- haskell head就是一个部分函数
    head [] //就会报错
    head [1,23] //1
    ```
### haskell 建立在currying的基础上，所有的函授其实都是只接受一个参数

### 类型类很像是接口

### 纯函数与不纯的函数加以区分

### 逻辑与IO分离（逻辑不应当依赖于外部状态，而应当只依赖于输入）

### cabal安装的包 必须使用 ghci -package package-name 才能在ghci中引用，若是此发生错误就把包删了重装，安装时出错也是一样，发现那个依赖不对就删掉重新安装


### 群的概念 google到数学里定义的群(group): G为非空集合，如果在G上定义的二元运算 *，满足

（1）封闭性（Closure）：对于任意a，b∈G，有a*b∈G
（2）结合律（Associativity）：对于任意a，b，c∈G，有（a*b）*c=a*（b*c）
（3）幺元 （Identity）：存在幺元e，使得对于任意a∈G，e*a=a*e=a
（4）逆元：对于任意a∈G，存在逆元a^-1，使得a^-1*a=a*a^-1=e
则称（G，*）是群，简称G是群。

如果仅满足封闭性和结合律，则称G是一个半群（Semigroup）；如果仅满足封闭性、结合律并且有幺元，则称G是一个含幺半群（Monoid）。

### => 之前的括号内是用来想定typeclass的，类型直接在方法中限制就可以了
```
pl :: (Show a, Num a) => Tree a -> Tree a
//typeclass中不但可以直接定义方法也是可以写的

class SemiGroup a where
  append :: a -> a -> a
  identiy :: a

instance SemiGroup Integer where
  append a b = a + b
  identiy = 0
```

### typeclass list https://wiki.haskell.org/List_instance

### haskell 语言规范 

### 结合律的并行能力

### Functor & Applicative & Monoid & Monad
```
--Functor typeclass
class Functor f where
  fmap :: (a -> b) -> f a -> f b
 
  (<$) :: a        -> f b -> f a
  (<$) = fmap . const

--Applicative typeclass
class (Functor f) => Applicative f where  
    pure :: a -> f a  
    (<*>) :: f (a -> b) -> f a -> f b

--Monoid typeclass
class Applicative m => Monad m where
  return :: a -> m a
  (>>=)  :: m a -> (a -> m b) -> m b
  (>>)   :: m a -> m b -> m b
  m >> n = m >>= \_ -> n
 
  fail   :: String -> m a 
--
--

```

### https://stackoverflow.com/questions/29154049/how-to-understand-type-inference-of-ap-liftm2-id
从这篇文章中我们可以看到haskell的函数签名时可以任意组合的
还有这份对(a -> b)的Functor的实现 https://downloads.haskell.org/~ghc/latest/docs/html/libraries/base-4.10.0.0/src/GHC-Base.html#%3C%24
也是类似的， 从上面的代码中我们可以看到mappend的签名是 mappend :: a -> a -> a
但我们可以看到当为(a -> b)这个Monoid实现时 mappend的实现是
mappend f g x = f x `mappend` g x， 问题来了为什么多了一个参数x呢，很简单，我们看到mappend的签名是类型变量，也就是说a可以使任意变量，是要变量时类型类Monoid的实现即可， 那我们把（a -> b）带入到签名即可看到 mappend的签名就成了， (a -> b) -> (a -> b) -> (a -> b), 当我们把最后一个括号去掉就可以看到签名变成了 (a -> b) -> (a -> b) -> a -> b,很显然现在就明显了，我们可以把mappend的签名看成需要三个参数了，最后返回b