---
title: Lambda 表达式有何用处？如何使用？  
date: 2018-04-7 
tags:
- Java  
- Lambda     
---
作者：Mingqi  
链接：https://www.zhihu.com/question/20125256/answer/324121308  
来源：知乎  


### **什么是Lambda?**  
我们知道，对于一个Java变量，我们可以赋给其一个**“值”**。  
```java
int aValue = 129;
String aString = "Hello World";
Boolean aBoolean = aString.startWith("H");
```
如果你想把**“一块代码”**赋给一个Java变量，应该怎么做呢？  
比如，我想把右边那块代码，赋给一个叫做aBlockOfCode的Java变量：  
![](https://xly2018.github.io/long/img/lambda/v2-1cc87e82fba0872c2cae3fee08e8fe41_hd.jpg)  
在Java 8之前，这个是做不到的。但是Java 8问世之后，利用Lambda特性，就可以做到了。  
![](https://xly2018.github.io/long/img/lambda/v2-145a556d86806c3163391a13428e3f03_hd.jpg) 
当然，这个并不是一个很简洁的写法。所以，为了使这个赋值操作更加elegant, 我们可以移除一些没用的声明。  
![](https://xly2018.github.io/long/img/lambda/v2-a712753b42972e094a548ae02fa82987_hd.jpg) 
这样，我们就成功的非常优雅的把“一块代码”赋给了一个变量。**而“这块代码”，或者说“这个被赋给一个变量的函数”，就是一个Lambda表达式**。  

但是这里仍然有一个问题，就是变量aBlockOfCode的类型应该是什么？ 

在Java 8里面，**所有的Lambda的类型都是一个接口，而Lambda表达式本身，也就是”那段代码“，需要是这个接口的实现**。这是我认为理解Lambda的一个关键所在，简而言之就是，**Lambda表达式本身就是一个接口的实现**。直接这样说可能还是有点让人困扰，我们继续看看例子。我们给上面的aBlockOfCode加上一个类型：  
![](https://xly2018.github.io/long/img/lambda/v2-55de66060b4cb70193ddc7fea201b257_hd.jpg) 
这种只有**一个接口函数需要被实现的接口类型，我们叫它”函数式接口“**。为了避免后来的人在这个接口中增加接口函数导致其有多个接口函数需要被实现，变成"非函数接口”，我们可以在这个上面加上一个声明@FunctionalInterface, 这样别人就无法在里面添加新的接口函数了：  
![](https://xly2018.github.io/long/img/lambda/v2-2c57e7411de227d1eb09c327d01fb766_hd.jpg)  
这样，我们就得到了一个完整的Lambda表达式声明：  
![](https://xly2018.github.io/long/img/lambda/v2-02eedc528fcee115f5ed0b7b045846d7_hd.jpg) 
###  **Lambda表达式有什么作用?** 
最直观的作用就是使得代码变得异常简洁。我们可以对比一下Lambda表达式和传统的Java对同一个接口的实现：  
![](https://xly2018.github.io/long/img/lambda/v2-dbd46cf9d188d0fde25db700c23dcc79_hd.jpg)
这两种写法本质上是等价的。但是显然，Java 8中的写法更加优雅简洁。并且，由于Lambda可以直接赋值给一个变量，**我们就可以直接把Lambda作为参数传给函数, 而传统的Java必须有明确的接口实现的定义，初始化才行：**  
![](https://xly2018.github.io/long/img/lambda/v2-28606f4328308baf7f70a36bd689e5ea_hd.jpg)
有些情况下，这个接口实现只需要用到一次。传统的Java 7必须要求你定义一个“污染环境”的接口实现MyInterfaceImpl，而相较之下Java 8的Lambda, 就显得干净很多。  


**Lambda结合FunctionalInterface Lib, forEach, stream()，method reference等新特性可以使代码变的更加简洁！**  
直接上例子。  
假设Person的定义和List<Person>的值都给定。  
![](https://xly2018.github.io/long/img/lambda/v2-02ae14b4c341e2303ae86e2da6a038a2_hd.jpg) 
现在需要你打印出guiltyPersons List里面所有LastName以"Z"开头的人的FirstName。  
**原生态Lambda写法：**定义两个函数式接口，定义一个静态函数，调用静态函数并给参数赋值Lambda表达式。  
![](https://xly2018.github.io/long/img/lambda/v2-fdef41934be8804fa244e89c84a567f1_hd.jpg)

这个代码实际上已经比较简洁了，但是我们还可以更简洁么？  

当然可以。在Java 8中有一个函数式接口的包，里面定义了大量可能用到的函数式接口（java.util.function (Java Platform SE 8 )）。所以，我们在这里压根都不需要定义NameChecker和Executor这两个函数式接口，直接用Java 8函数式接口包里的Predicate<T>和Consumer<T>就可以了——因为他们这一对的接口定义和NameChecker/Executor其实是一样的。  
![](https://xly2018.github.io/long/img/lambda/v2-ad5b3e8d225bf9e6d988c6dc83819637_hd.jpg) 

**第一步简化 - 利用函数式接口包：**  

![](https://xly2018.github.io/long/img/lambda/v2-493168d1a72120d69b0d13b4711c3c9f_hd.jpg) 
静态函数里面的for each循环其实是非常碍眼的。这里可以利用Iterable自带的forEach()来替代。forEach()本身可以接受一个Consumer<T> 参数。  

**第二步简化 - 用Iterable.forEach()取代foreach loop：**

![](https://xly2018.github.io/long/img/lambda/v2-bdd6f5a6dbf65b578080f7b7ad5ab6cf_hd.jpg)  
由于静态函数其实只是对List进行了一通操作，这里我们可以甩掉静态函数，直接使用stream()特性来完成。stream()的几个方法都是接受Predicate<T>，Consumer<T>等参数的（java.util.stream (Java Platform SE 8 )）。你理解了上面的内容，stream()这里就非常好理解了，并不需要多做解释。  

**第三步简化 - 利用stream()替代静态函数：**

![](https://xly2018.github.io/long/img/lambda/v2-e196d987f852b9b8e26a6a9dac648a06_hd.jpg)
对比最开始的Lambda写法，这里已经非常非常简洁了。但是如果，我们的要求变一下，变成print这个人的全部信息，及p -> System.out.println(p); 那么还可以利用Method reference来继续简化。所谓Method reference, 就是用已经写好的别的Object/Class的method来代替Lambda expression。格式如下：  

![](https://xly2018.github.io/long/img/lambda/v2-12622326a5682285ce235d96291f3bb8_hd.jpg) 

**第四步简化 - 如果是println(p)，则可以利用Method reference代替forEach中的Lambda表达式：**  
![](https://xly2018.github.io/long/img/lambda/v2-f29e6569d0265b91794565ae81d54265_hd.jpg)
这基本上就是能写的最简洁的版本了。  

 
**Lambda配合Optional<T>可以使Java对于null的处理变的异常优雅**  
这里假设我们有一个person object，以及一个person object的Optional wrapper:  
![](https://xly2018.github.io/long/img/lambda/v2-e78d71b9699504e46d86d77aabec42cf_hd.jpg) 
Optional<T>如果不结合Lambda使用的话，并不能使原来繁琐的null check变的简单。  
![](https://xly2018.github.io/long/img/lambda/v2-ce82e2c3e69caab9acdec9d4e42cde0e_hd.jpg) 
**只有当Optional<T>结合Lambda一起使用的时候，才能发挥出其真正的威力！**  

我们现在就来对比一下下面四种常见的null处理中，Java 8的Lambda+Optional<T>和传统Java两者之间对于null的处理差异。
  
**情况一 - 存在则开干**  

![](https://xly2018.github.io/long/img/lambda/v2-f41305009c93effa8fe047631d5342ed_hd.jpg) 

**情况二 - 存在则返回，无则返回屁**  

![](https://xly2018.github.io/long/img/lambda/v2-cfa0e3298ba94efa29dc9cb8b32356fe_hd.jpg)

**情况三 - 存在则返回，无则由函数产生**  

![](https://xly2018.github.io/long/img/lambda/v2-4091bb966ac575fd83d5fa07dd7c2dce_hd.jpg)

**情况四 - 夺命连环null检查**  

![](https://xly2018.github.io/long/img/lambda/v2-7bc0ac86cd29a1cea92a853d73f41cec_hd.jpg) 
由上述四种情况可以清楚地看到，Optional<T>+Lambda可以让我们少写很多ifElse块。尤其是对于情况四那种夺命连环null检查，传统java的写法显得冗长难懂，而新的Optional<T>+Lambda则清新脱俗，清楚简洁。