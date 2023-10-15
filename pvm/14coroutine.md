# 深入理解 Python 虚拟机：协程初探——不过是生成器而已

在 Python 3.4 Python 引入了一个非常有用的特性——协程，在后续的 Python 版本当中不断的进行优化和改进，引入了新的 await 和 async 语法。在本篇文章当中我们将详细介绍一下 Python 协程的原理以及虚拟机具体的实现协程的方式。

## 什么是协程

Coroutines are computer program components that allow execution to be suspended and resumed, generalizing subroutines for cooperative multitasking. 

根据 wiki 的描述，协程是一个允许停下来和恢复执行的程序，从文字上来看这与我们的常识或者直觉是相互违背的，因为在大多数情况下我们的函数都是执行完才返回的。其实目前 Python 当中早已有了一个特性能够做到这一点，就是生成器，如果想深入了解一下生成器的实现原理和相关的字节码可以参考这篇文章 [深入理解 Python 虚拟机：生成器停止背后的魔法](https://github.com/Chang-LeHung/dive-into-cpython/blob/master/pvm/10generator.md) 。

现在在 Python 当中可以使用 async 语法定一个协程函数（当函数使用 async 进行修饰的时候这个函数就是协程函数），当我们调用这个函数的时候会返回一个协程对象，而不是直接调用函数：

```python
>>> async def hello():
...     return 0
... 
>>> hello()
<coroutine object hello at 0x100a04740>
```

在 inspect 模块当中也有一个方法用于判断一个函数是否是协程函数：

```python
import inspect

async def hello():
	return 0

print(inspect.iscoroutinefunction(hello)) # True
```

在 Python 当中当你想创建一个协程的话，就直接使用一个 async 关键字定一个函数，调用这个函数就可以得到一个协程对象。

在协程当中可以使用 await 关键字等待其他协程完成，当被等待的协程执行完成之后，就会返回到当前协程继续执行：

```python 
import asyncio
import datetime
import time


async def sleep(t):
	time.sleep(t)


async def hello():
	print("start a coroutine", datetime.datetime.now())
	await sleep(3)
	print("wait for 3s", datetime.datetime.now())


if __name__ == '__main__':
	coroutine = hello()
	try:
		coroutine.send(None)
	except StopIteration:
		print("coroutine finished")
```

```bash
start a coroutine 2023-10-15 02:21:33.503505
wait for 3s 2023-10-15 02:21:36.503984
coroutine finished
```

在上面的程序当中，await asyncio.sleep(3) 确实等待了 3 秒之后才继续执行。

## 协程的实现

在 Python 当中协程其实就是生成器，只不过在生成器的基础之上稍微包装了一下，比如在写成当中的 await 语句，其实作用和 yield from 对于生成器的作用差不多，稍微有点细微差别。我们用几个例子来详细分析一下协程和生成器之间的关系：

```python
async def hello():
	return 0

if __name__ == '__main__':
	coroutine = hello()
	print(coroutine)
	try:
		coroutine.send(None)
	except StopIteration:
		print("coroutine finished")
```

上面的代码的输出结果：

```bash
<coroutine object hello at 0x1170200c0>
coroutine finished
```

在上面的代码当中首先调用 hello 之后返回一个协程对象，协程对象和生成器对象一样都有 send 方法，而且作用也一样都是让协程开始执行，然后写成直接执行完成。和生成器一样当一个生成器执行完成之后会产生 StopIteration 异常，因此需要对异常进行 try catch 处理。

当然你也可以使用 yield 语句让协程的执行停下来：

```python
```


