---
layout: post
title: R代码阅读随笔
categories:
- R
tags:
- R
- 源码
- 随笔
---


开始学习R，从阅读源码开始吧。
不得不说R的语法很让人费解。虽然有[r-language][r-lang]文档作为辅助说明，但还是让熟悉Java，Python等语言的程序员无法理解R中那些让人不得不吐槽的语法，个人觉得，如果R的语法能如MathLab那样，那么R将会有更多的受众。

---
##1.base
---
###A
-----

####all.equal.R
all.equal的第一行语法就不是很明了，这个UseMethod时用来做什么的？
看UseMethod的文档还是云里雾里的。
>all.equal <- function(target, current, ...) UseMethod("all.equal")

当该方法返回为TRUE时，说明两个比较对象相等，返回其他的值时就意味着不等了。

在**all.equal.default**中看到switch的用法，确实很神奇，按照r-language中的解释，switch(statement,list),或者switch(statement,...)先对statement求值，如果返回值是一个数字，那么取得list[value]对应的expression进行求值并返回，如果value是一个字符串，那么...中与之匹配的name=expression中的name对应的expression将会被求值并返回，如果没有匹配的，看是否还有default子句，如果有就返回default的求值，否则返回NULL。如：

>y <- "fruit"

>f <- switch(y,fruit="banana",vegetable="broccoli","Unknown")

>结果就是f == "banana"

在R中向量或者列表的index操作就比较丰富了，各种切片(slice)，还有那些list[!booleans],list[-index],在学python时以为python的下标操作就够灵活的了，这一个简直就秒杀python了。
下面的代码比较神奇，不过我喜欢（这难道就是函数式编程的魅力？？？）
> x <- TRUE ## FALSE

>(if(x) c else sum)(1,2)

这是R的特点还是普遍的函数式编程的特点呢？这里我给不出答案，毕竟按照编程的年龄来算，我还是个新手。
在***all.equal.language*中，出现的paste0,和之前出现的paste的差别在于paste0连接的字符串之间时没有间隔的，而paste默认连接的字符串之间有间隔，当然可以通过设置paste方法中的sep参数来指定分割符。文档中说时paste0效率稍微高一点，其他的就等效于paste(x,sep="")。

all.equal.R中的attr.all.equal就是一个递归的调用all.equal了，这样就可以进行更深层次的比较了。

####all.names.R
all.names方法的实现类似于Java中JNI的那种方式吧(自认为是这样的，也不知道是否恰当)，把具体实现交给了C，这样效率更高了。调用的模式就如下:
>all.names <- function(expr, functions = TRUE, max.names = -1L, unique = FALSE)

>.Internal(all.names(expr, functions, max.names, unique))

文档中说时获取expression中的所有names，做的事情应当就是把一个expression分解成基本的符号单位(如sin(x+y)就会被分解成“sin”，“+”，“x”，“y”，具体分解的顺序应当是按照语法树的结构)。现在还没有研究过.Internal的机制，等以后在说吧。

all.vars方法就是只抽取出expression中的变量名字，而把方法名忽略（如上例的"sin"，"+"）
####aperm.R
这个方法对一个array做转置，但是对其中的perm这个参数不是很明了，在高维数组中，转置是如何定义的？其中perm的取值范围和array的dim时有关系的，具体什么关系也没有高清楚。这个方法需要传入一个array对象，在创建array时，一不小心把array中dim这个参数vector设置的大了些，电脑立马就处于full-busy的状态了。
####append.R
按照给定的after参数，向已知list中添加元素（按照代码的实现，返回的list对象已经不是原来的对象了(作为一个Java或者Python用户，这里虽然时append一个元素，但是具体的实现却是像Java中的字符串拼接一样，得到的字符串是一个新的对象)）。用法和Python中的不一样，按照R中该方法给定的参数，我觉得这个方法名叫做insert更为合适，因为它指定了插入元素的位置。
####apply.R
这个方法在很多R语言的书中都有介绍(apply方法簇)，按照文档上说的，该方法apply(x,margins,fun,...)，按照margins指定的顺序对x应用fun[ction],但对margins这个参数的用法不太明白。文档中说当margins=1时表明是row，2表明时column，c(1,2)表明时row和column。如下：
>f <- function(x) x+1

>x <- matrix(1:9,c(3,3))

>r <- apply(x,1,f)

>c <- apply(x,2,f)

得到的结果是c和r互为转置。按照文档中的说明，margin的不同只是f应用于x的顺序不同。但是这里明显是有多余操作的：当f应用于x的每个元素生成返回矩阵时，是按行生成的，而并没有遵循：x某个位置的元素在应用了f后生成返回矩阵的位置应当和在x中的位置一致。其中这一行newX <- aperm(X, c(s.call, s.ans))，对margin参数为1时，对x进行了转置。当margin为2时，保持矩阵的样式未变。

####array.R
array()这个方法每什么可说的，但**array.R**中有个slice.index，这个方法的说明也让我挺费解的，反正我暂时没有搞明白这个方法的用处。

####as.R
这个脚本里面也没什么可说的，就是一堆转换方法，倒是最后一个as.qr给的挺无赖的，
>as.qr <- function(x) stop("you cannot be serious")

程序员偶尔也是需要牢骚下的。

####assign.R
assign就没什么可说的，相当于 <- 这个赋值符号，不过通过assign这个方法可以指定environment，但 <- 的操作却只能作用于当前environment。关于environment，我的理解就相当于Java或者Python中域（scope）的概念吧，在这个域中，维持了一堆当前的变量，方法等等，当然还有一个指向父域的指针，这里父域和子域的关系，就相当于Java中类的继承关系吧，在Java的继承体系中，方法，变量的寻找，首先时在子类中进行的，找不到才取父类中寻找，这里的environment也有类似的概念。

该文件中还有一个list2env方法，这个方法就是拥一个list的对象来初始化一个environment，当然list中时什么都可以有的，比如变量，方法，等等，这样就可以把enviroment初始化了。默认情况下，该environment时作为当前域的兄弟节点的，因为在默认情况下，该域的parent参数是设置成当前域的parent的。

####attach.R
在几本关于R的书上都看到过关于attach的介绍，但最后都会有一个建议，即不建议使用attach方法。这个方法的作用，就像在Java中更改Java中包的搜索顺序一样，比如一个简单的例子，JDK中有一个类，我自己些的package中也有一个类，他们的名字及包路径都是相同的，默认情况下，Java优先搜索JDK中的方法，但是我人为的设置，使得Java优先搜索我自己的package，这样就会首先找到我自己的package中定义的类了。这里的attach方法，就是把一些额外的library或者如文档里所谓的“database”添加到搜索路径中，当然如果一个人进行R程序开发，这只要记得及时unattach掉这些database就Ok了，但是如果是多人协作开发，那么就会有很大的问题了，就像在C里面，路人甲申请了一段内存，而他并没有释放掉，却寄希望于一块开发程序的路人乙、丙或者丁来帮他释放掉，这太危险了。

当然attach.R中也提供了detach这个方法来释放掉attach的database，但还是不建议在程序中使用attach，用文档中的话说就是side effect比较多

看到这里，感觉R中任何语句都时可以求值的，这一点很强悍，比如：
> x <- if(bool) "TRUE" else "FALSE"

而在其他如Python，虽然同为脚本语言，却没有这样的特性。

####attr.R
这里面又是一个神奇的用法，
>\`mostattributes<-\` <- function (x,...)

这里需要注意的时键盘上～这个符号,这才是英语的连的开引号(\`)，当然还有闭引号(')，必须承认在以前的文档写作时从来没有注意这种引号的区别，直到前几天在Letax中才看到，原来英文的引号是有开闭之分的（在中文中当然是有这种区别的了），上面只是对引号的觉得奇怪，更奇怪的应当是这种语法吧，按照我现在的理解，这种方法的用法可以是这样子的：
>mostatributes(x) <- value

这对熟悉Java或者Python的用户来说实在时要惊艳下了。

----
###B
----
####backquote.R
拥指定域中的变量替换表达式中的变量引用，被引用的变量一般使用".(var)"来声明。比如:
>a <- "abc"
>
bquote(b == .(a))

则输出为b==“abc”，源码中先判断是否为pairlist，然后在查看pairlist的文档时，又看到一段神奇的代码
> f <- function() x
>
>formals(f) <- al <- alist(x=,y=2+3,...=)

这样，f这个方法的定义就变成下面这个样子了：
> function(x,y=2+3,...) x

熟悉Java和Python人要咋舌了吧，程序的世界无奇不有，快从Java和Python的小圈子中跳出来拥抱R吧。

####backsolve.R
该脚本中包含两个方法，forwordsolve。backsolve，是用来解线性方程组的比如R×X=b，但对该线性方程组的系数矩阵做了假设，即R为**上三角矩阵**或者**下三角矩阵**，就算在调用这两个方法时传入的R参数并不是三角矩阵，那么在方法的实现内部也会对R做调整，调整为uper.tri参数所指定类型的三角阵。当然了像解方程组这样大计算量的动作，R会毫不犹豫的委托给C或者其他的效率更高的实现了。这里就是使用了.C这个函数来调用外部接口的。具体.C的用法如何，还要等到后面来观察。

####Bessel.R
[贝塞尔函数][bessel]，实现了贝塞尔函数第一类，第二类和变形贝塞尔函数。这里不做具体解释，因为我也时不懂。

####bindenv.R
这种对域、变量进行锁定，解锁的操作，从之前的编程经验上来说，确实很神奇，这样极大的增强了程序员对程序的控制能力。但这种操作，对于并发性编程，是机遇还是挑战呢？以我现在对R的了解程度来说，我无法给出答案。

####builtins.R
返回所有的内建对象。当然这里的内建对象是symbol table中的，我们可以通过传递internal参数来指定是不是返回通过.internal方法可以使用的对象。

####by.R
类似于apply函数簇，按照文档上的说明，by时对tapply的封装。而在其中看到这样一个还算时比较值得记忆的应用方式
>data
>
>FUN
>
>FUNx <function(x) Fun(data[x,])
>
>NumofRows <- nrow(data)
>
>tapply(seq_len(NumofRows),IND,Funx)

观察Funx的声明，把data作为内部变量使用了，像Python闭包中的用法吧。

----
###C
---
####callCC.R
这个脚本里的方法本身很简单，就是提供了一种程序退出机制。按照文档中的说明，是一种非本地退出机制，按照我的理解，一般的退出机制是当方法执行到结束或者异常引起的，但这里的退出机制却是指定了只要一执行完某个函数或者某行代码，就自动退出了。比如：
>callCC(function(k) repeat k(1))

只要k(1)这个方法一执行，就自动退出了。这里的退出就是使用了return的方式，就像我们在这个k(1)这个方法内插入了一条return语句一样。

####cat.R
一种格式化的print方法，当然这里的print并不是说只能在控制台打印，这里可以打印到文件中，和unix或者linux中的cat命令有相似之处。cat方法不如print方法方便，但提供了更多的灵活性。

####character.R
字符串操作方法，这里需要注意substr和substring的区别
substr接受start和stop参数，用来指定子串的开始和结束点，方法内是把start和stop强制转换成integer处理的。而substring方法却是内藏玄机。首先substring方法中的first和last参数不一定是integer，还可能时vector等类型，比如
>start <- c(1,2)
>
>last <- c(3,4,5)
>
>substring("ABCDEF",start,stop)

上面代码返回的结果时这样计算出来的，首先把start和last转变成等长vector，具体的如何转变，参照[r-lang][r-lang]中Recycling-rules这一小节。然后依次按照start和stop中元素位置顺序从字符串中取出start和stop的位置之间的字串。当然，代码中并不是像我上面说的这样子实现的，代码中是把字符串rep后再像substr那样子处理的。其本质区别就在于substring中对字符串进行了rep处理，而substr中没有进行。
其中的abbreviate方法，不能算是一个可以得到很好结果的方法，毕竟在英语中缩写的规律并不是有很明了的规定。
###chol.R
R中chol的具体实现是交给Fortran实现的，暂时未看具体实现代码。这里仅对一些常用的矩阵分解做介绍，这里的讨论仅限于实数域。

Cholesky分解,对一个实对称正定方阵进行分解 A = L^t * L,
其中L为一个上三角阵，L^t 是对L的转置。下面是一个简单的chol的R代码实现。
{%highlight python %}
###参数x必须是一个实对称正定阵，返回一个上三角阵
chol_demo <- function(x){
  if(is.matrix(x)){
    d <- dim(x)
    l <- matrix(rep(0,d[1]*d[2]),nrow=d[1],ncol=d[2])
    for(i in seq_len(d[1])){
      l[i,i] <- sqrt(x[i,i] - sum((l[i,seq_len(i-1)])^2))
      if(i<d[1]){
        for(j in seq(i+1,d[1])){
          l[j,i] =(x[i,j]-sum(l[i,seq_len(i-1)]*l[j,seq_len(i-1)]))/l[i,i]
        } 
      }
    }
    t(l)
  }else{
    stop("x is not a matrix")
  }
}
{% endhighlight %}
脚本里面的另一个方法chol2inv，按照chol.R的定义A=L^t * L,其中L为上三角阵。那么chol2inv做的事情就如下：chol2inv(L) = A^-1 ,即chol2inv根据L求得A的逆。

-----
[r-lang]: http://cran.r-project.org/doc/manuals/R-lang.html
[bessel]: http://zh.wikipedia.org/wiki/贝塞尔函数


