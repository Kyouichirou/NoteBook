
# streamlit缓存设置
## Optimize performance with st.cache

#### Important

We're developing new cache primitives that are easier to use and much faster than `@st.cache`. 🚀 To learn more, read [Experimental cache primitives](https://docs.streamlit.io/library/advanced-features/experimental-cache-primitives).

我们开发了新的缓存机制(primitives), 使用更容易, 而且速度比`@st.cache`速度更快, 了解更多, 请参考相关页面(试验中).

Streamlit provides a caching mechanism that allows your app to stay performant even when loading data from the web, manipulating large datasets, or performing expensive computations. This is done with the [`@st.cache`](https://docs.streamlit.io/library/api-reference/performance/st.cache) decorator.

streamlit提供的缓存机制能为你的app在数据加载, 操作大型数据集或者其他需要耗费算力的操作中保持高效. 只需要使用`@st.cache`装饰器.

When you mark a function with the [`@st.cache`](https://docs.streamlit.io/library/api-reference/performance/st.cache) decorator, it tells Streamlit that whenever the function is called it needs to check a few things:

当你在函数中使用`@st.cache`装饰器, 这意味告诉streamlit在函数调用时, 需要做一些检查:

1. The input parameters that you called the function with
2. The value of any external variable used in the function
3. The body of the function
4. The body of any function used inside the cached function

- 输入的参数
- 函数中使用的任意外部变量
- 函数主体
- 已缓存函数使用的函数主体

If this is the first time Streamlit has seen these four components with these exact values and in this exact combination and order, it runs the function and stores the result in a local cache. Then, next time the cached function is called, if none of these components changed, Streamlit will just skip executing the function altogether and, instead, return the output previously stored in the cache.

如果这是streamlit第一次运行, streamlit将会缓存运行的结果到本地缓存. 下次再运行该函数时, 假如上述提及的四个部分没有发生变化, streamlit将不会执行函数, 而是直接返回之前缓存中的结果.

The way Streamlit keeps track of changes in these components is through hashing. Think of the cache as an in-memory key-value store, where the key is a hash of all of the above and the value is the actual output object passed by reference.

streamlit通过hash值来判断这些内容是否发生变化. 可以认为缓存在内存中以键值对的形式存在, 键是上述内容的的hash值, 值是实际输出结果对象的引用.

Finally, [`@st.cache`](https://docs.streamlit.io/library/api-reference/performance/st.cache) supports arguments to configure the cache's behavior. You can find more information on those in our [API reference](https://docs.streamlit.io/library/api-reference).

同时, `@st.cache`支持参数来配置缓存的行为, 更新信息见API参考.

Let's take a look at a few examples that illustrate how caching works in a Streamlit app.

看一下具体的例子在实际中的使用

注: 官方文档下面的案例,  把一些简单的问题解释的过于复杂难懂, 不需要完全看官方的解释.

## st.cache

Function decorator to memoize function executions.

| Function signature                                           |                                                              |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| st.cache(func=None, persist=False, allow_output_mutation=False, show_spinner=True, suppress_st_warning=False, hash_funcs=None, max_entries=None, ttl=None) |                                                              |
| Parameters                                                   |                                                              |
| **func** *(callable)*                                        | The function to cache. Streamlit hashes the function and dependent code. 需要缓存的函数 |
| **persist** *(boolean)*                                      | Whether to persist the cache on disk. 持久化(目前只支持一个选项, 持久化到硬盘) |
| **allow_output_mutation** *(boolean)*                        | Streamlit shows a warning when return values are mutated, as that can have unintended consequences. This is done by hashing the return value internally.If you know what you're doing and would like to override this warning, set this to True. 可变对象输出警告 |
| **show_spinner** *(boolean)*                                 | Enable the spinner. Default is True to show a spinner when there is a cache miss. 缓存丢失 |
| **suppress_st_warning** *(boolean)*                          | Suppress warnings about calling Streamlit functions from within the cached function. 忽略调用缓存函数警告 |
| **hash_funcs** *(dict or None)*                              | Mapping of types or fully qualified names to hash functions. This is used to override the behavior of the hasher inside Streamlit's caching mechanism: when the hasher encounters an object, it will first check to see if its type matches a key in this dict and, if so, will use the provided function to generate a hash for it. See below for an example of how this can be used. 哈希函数, 可以自定执行哈希的函数 |
| **max_entries** *(int or None)*                              | The maximum number of entries to keep in the cache, or None for an unbounded cache. (When a new entry is added to a full cache, the oldest cached entry will be removed.) The default is None.缓存数量 |
| **ttl** *(float or None)*                                    | The maximum number of seconds to keep an entry in the cache, or None if cache entries should not expire. The default is None.缓存保持时间 |

## Example 1: Basic usage

案例1: 基础使用

For starters, let's take a look at a sample app that has a function that performs an expensive, long-running computation. Without caching, this function is rerun each time the app is refreshed, leading to a poor user experience. Copy this code into a new app and try it out yourself:

起始, 先看一个需要花费时间的行为, 没有缓存, 每一次app页面刷新, 这个函数都需要重新执行一遍返回结果, 这对于用户体验而言是相当糟糕的. 复制下面代码, 亲自尝试一下吧.

```python
import streamlit as st
import time

# 模拟一个需要耗费时间的操作
def expensive_computation(a, b):
    time.sleep(2)  # 👈 This makes the function take 2s to run
    return a * b

a = 2
b = 21
res = expensive_computation(a, b)

st.write("Result:", res)
```

Try pressing **R** to rerun the app, and notice how long it takes for the result to show up. This is because `expensive_computation(a, b)` is being re-executed every time the app runs. This isn't a great experience.

尝试按下`R`返回app, 会注意到页面需要花费很长实现来呈现结果. 这是因为每次的执行都是重新运行一次函数, 这是非常花费时间的, 这不是很好的体验.

Let's add the [`@st.cache`](https://docs.streamlit.io/library/api-reference/performance/st.cache) decorator:

添加装饰器

```python
import streamlit as st
import time

@st.cache  # 👈 Added this
def expensive_computation(a, b):
    time.sleep(2)  # This makes the function take 2s to run
    return a * b

a = 2
b = 21
res = expensive_computation(a, b)

st.write("Result:", res)
```

Now run the app again and you'll notice that it is much faster every time you press R to rerun. To understand what is happening, let's add an st.write inside the function:

现在再次运行app, 你会发现运行的速度非常快. 想了解发生了什么嘛, 增加一个`st.write`到函数去.

```python
import streamlit as st
import time

@st.cache(suppress_st_warning=True)  # 👈 Changed this
def expensive_computation(a, b):
    # 👇 Added this
    st.write("Cache miss: expensive_computation(", a, ",", b, ") ran")
    time.sleep(2)  # This makes the function take 2s to run
    return a * b

a = 2
b = 21
res = expensive_computation(a, b)

st.write("Result:", res)
```

Now when you rerun the app the text "Cache miss" appears on the first run, but not on any subsequent runs. That's because the cached function is only being executed once, and every time after that you're actually hitting the cache.

现在当你运行这个app时, "Cache miss"这部分的内容只会出现在第一次运行时出现. 这是因为缓存的功能, 这个函数只执行了一遍.

#### Note

注意

You may have noticed that we've added the `suppress_st_warning` keyword to the `@st.cache` decorators. That's because the cached function above uses a Streamlit command itself (`st.write` in this case), and when Streamlit sees that, it shows a warning that your command will only execute when you get a cache miss. More often than not, when you see that warning it's because there's a bug in your code. However, in our case we're using the `st.write` command to demonstrate when the cache is being missed, so the behavior Streamlit is warning us about is exactly what we want. As a result, we are passing in `suppress_st_warning=True` to turn that warning off.

你也许已经注意到我们添加`suppress_st_warning`关键字到`@st.cache`装饰器上. 这是因为上述缓存的函数使用了streamlit的st.write(), 当streamlit检测到这个存在, 只有当你遇到缓存丢失时, 你只有执行代码时它会展示一个警告. 通常情况下, 当你看到这个警告, 这意味着你的代码存在bug. 

## Example 2: When the function arguments change

案例2: 当函数参数改变

*注: 官方文档解释这个问题很啰嗦混乱, 简单而言, `suppress_st_warning=True`就是用来空值缓存丢失警告用的.*

*当你修改了函数相关的代码, 就会出现缓存丢失的问题, 如果不希望出现警告, 就是用这个flag.*

Without stopping the previous app server, let's change one of the arguments to our cached function:

没有停止上述的app服务器, 修改缓存函数其中的一个参数.

```python
import streamlit as st
import time

@st.cache(suppress_st_warning=True)
def expensive_computation(a, b):
    st.write("Cache miss: expensive_computation(", a, ",", b, ") ran")
    time.sleep(2)  # This makes the function take 2s to run
    return a * b

a = 2
b = 210  # 👈 Changed this
res = expensive_computation(a, b)

st.write("Result:", res)
```

Now the first time you rerun the app it's a cache miss. This is evidenced by the "Cache miss" text showing up and the app taking 2s to finish running. After that, if you press **R** to rerun, it's always a cache hit. That is, no such text shows up and the app is fast again.

当你第一次运行时, 上述文本会花费2秒来展示这文本. 当你重新按下`R`键时, 这个文本不会展示, 因为缓存.

This is because Streamlit notices whenever the arguments **a** and **b** change and determines whether the function should be re-executed and re-cached.

## Example 3: When the function body changes

案例3: 当函数的主体发生改变

Without stopping and restarting your Streamlit server, let's remove the widget from our app and modify the function's code by adding a `+ 1` to the return value.

没有停止和重启streamlit服务器, 移除这个组件从app当中

```python
import streamlit as st
import time

@st.cache(suppress_st_warning=True)
def expensive_computation(a, b):
    st.write("Cache miss: expensive_computation(", a, ",", b, ") ran")
    time.sleep(2)  # This makes the function take 2s to run
    return a * b + 1  # 👈 Added a +1 at the end here

a = 2
b = 210
res = expensive_computation(a, b)

st.write("Result:", res)
```

The first run is a "Cache miss", but when you press **R** each subsequent run is a cache hit. This is because on first run, Streamlit detected that the function body changed, reran the function, and put the result in the cache.

#### Tip

提示

If you change the function back the result will already be in the Streamlit cache from a previous run. Try it out!

## Example 4: When an inner function changes

案例4: 当一个内部函数改变

Let's make our cached function depend on another function internally:

让我们在缓存一个函数, 其内部依赖于另一个函数.

注: 简单而言, 外部的函数修改, 也会引发缓存丢失的警告, 并不需要完整的包裹在装饰器中(代码不需要完全包裹在装饰的函数内).

```python
import streamlit as st
import time

def inner_func(a, b):
    st.write("inner_func(", a, ",", b, ") ran")
    return a * b

@st.cache(suppress_st_warning=True)
def expensive_computation(a, b):
    st.write("Cache miss: expensive_computation(", a, ",", b, ") ran")
    time.sleep(2)  # This makes the function take 2s to run
    return inner_func(a, b) + 1

a = 2
b = 210
res = expensive_computation(a, b)

st.write("Result:", res)
```

What you see is the usual:

你通常会看到

1. The first run results in a cache miss.
1. 第一次运行才出现这个提示
2. Every subsequent rerun results in a cache hit.
2. 之后的每次运行将不会有这个提示.

But now let's try modifying the `inner_func()`:

但是现在, 修改`inner_func()`函数

```python
import streamlit as st
import time

def inner_func(a, b):
    st.write("inner_func(", a, ",", b, ") ran")
    return a ** b  # 👈 Changed the * to ** here

@st.cache(suppress_st_warning=True)
def expensive_computation(a, b):
    st.write("Cache miss: expensive_computation(", a, ",", b, ") ran")
    time.sleep(2)  # This makes the function take 2s to run
    return inner_func(a, b) + 1

a = 2
b = 21
res = expensive_computation(a, b)

st.write("Result:", res)
```

Even though `inner_func()` is not annotated with [`@st.cache`](https://docs.streamlit.io/library/api-reference/performance/st.cache), when we edit its body we cause a "Cache miss" in the outer `expensive_computation()`.

尽管`inner_func()`没有使用装饰器, 当我们编辑它的主体时, 

That's because Streamlit always traverses your code and its dependencies to verify that the cached values are still valid. This means that while developing your app you can edit your code freely without worrying about the cache. Any change you make to your app, Streamlit should do the right thing!

这是因为streamlit总是遍历代码以及依赖, 用于校验缓存值是否依然有效. 这意味着你可以自由地编辑代码而不需要考虑缓存的问题(意思是当你修改了代码之后, 缓存就会失效, 而不需要担心缓存的影响, 例如你改了代码, 理应返回不一样的值, 但是担心缓存的影响).

注: 这意味着可以热编程, 即一边修改代码, 一边运行查看, 而不需要每次修改代码之后重新启动streamlit服务器.

Streamlit is also smart enough to only traverse dependencies that belong to your app, and skip over any dependency that comes from an installed Python library.

streamlit智能程度足以区分开自有代码的依赖和python原生库的依赖, 只遍历属于app自身的依赖, 会跳过一个已经安装的python库(第三方还是原生?)依赖.

## Example 5: Use caching to speed up your app across users

案例5: 使用缓存为不同的用户加速app

Going back to our original function, let's add a widget to control the value of `b`:

回到最初的函数, 增加一个组件来控制值`b`.

```python
import streamlit as st
import time

@st.cache(suppress_st_warning=True)
def expensive_computation(a, b):
    st.write("Cache miss: expensive_computation(", a, ",", b, ") ran")
    time.sleep(2)  # This makes the function take 2s to run
    return a * b

a = 2
b = st.slider("Pick a number", 0, 10)  # 👈 Changed this
# 变更这个缓存将丢失
res = expensive_computation(a, b)

st.write("Result:", res)
```

What you'll see:

- If you move the slider to a number Streamlit hasn't seen before, you'll have a cache miss again. And every subsequent rerun with the same number will be a cache hit, of course.
- 当移动滑动条改变数字时, 假如这个数字没有出现过,  缓存丢失将会再次出现.
- If you move the slider back to a number Streamlit has seen before, the cache is hit and the app is fast as expected.
- 出现过, 也就意味缓存的产生

In computer science terms, what is happening here is that [`@st.cache`](https://docs.streamlit.io/library/api-reference/performance/st.cache) is [memoizing](https://en.wikipedia.org/wiki/Memoization) `expensive_computation(a, b)`.

用计算机术语来描述, 这时候的装饰器实在记住这个`expensive_computation(a, b)`函数.

But now let's go one step further! Try the following:

1. Move the slider to a number you haven't tried before, such as 9.
2. Pretend you're another user by opening another browser tab pointing to your Streamlit app (usually at http://localhost:8501)
3. In the new tab, move the slider to 9.

Notice how this is actually a cache hit! That is, you don't actually see the "Cache miss" text on the second tab even though that second user never moved the slider to 9 at any point prior to this.

这种缓存是作用在全局的, 不管是一个浏览器开多个标签, 还是在不同的机器上访问, 相关的内容只要访问过, 缓存的产生就是作用在全局的.

This happens because the Streamlit cache is global to all users. So everyone contributes to everyone else's performance.

一人贡献全局的效率.

## Example 6: Mutating cached values

案例6: 可变中的缓存值

As mentioned in the [overview](https://docs.streamlit.io/library/advanced-features/caching#optimize-performance-with-stcache) section, the Streamlit cache stores items by reference. This allows the Streamlit cache to support structures that aren't memory-managed by Python, such as TensorFlow objects. However, it can also lead to unexpected behavior — which is why Streamlit has a few checks to guide developers in the right direction. Let's look into those checks now.

正如在概要中所提及的, streamlit通过引用来存储缓存. 这允许streamlit缓存支持不是python管理的内存结构(即该对象的内存不是python所管理的), 例如TensorFlow对象. 然而这会导致不可预测的行为, 这也是为什么streamlit.

Let's write an app that has a cached function which returns a mutable object, and then let's follow up by mutating that object:

让我们写一个app, 这有一个缓存函数, 这个函数会返回一个易变值.

```python
import streamlit as st
import time
# @st.experimental_memo, 这个装饰器, 其效果也是一样的
# 不传入这个参数, suppress_st_warning=True, 将会在界面上出现警告
# @st.experimental_memo, 不具有这个参数, 不会出现警告
@st.cache(suppress_st_warning=True)
def expensive_computation(a, b):
    st.write("Cache miss: expensive_computation(", a, ",", b, ") ran")
    time.sleep(2)  # This makes the function take 2s to run
    return {"output": a * b}  # 👈 Mutable object

a = 2
b = 21
res = expensive_computation(a, b)

st.write("Result:", res)

res["output"] = "result was manually mutated"  # 👈 Mutated cached value

st.write("Mutated result:", res)
```

When you run this app for the first time, you should see three messages on the screen:

- Cache miss (...)
- Result: {output: 42}
- Mutated result: {output: "result was manually mutated"}

No surprises here. But now notice what happens when you rerun you app (i.e. press **R**):

- Result: {output: "result was manually mutated"}
- Mutated result: {output: "result was manually mutated"}
- Cached object mutated. (...)

So what's up?

What's going on here is that Streamlit caches the output `res` by reference. When you mutated `res["output"]` outside the cached function you ended up inadvertently modifying the cache. This means every subsequent call to `expensive_computation(2, 21)` will return the wrong value!

这里发生的事情是streamlit缓存的结果是通过引用的方式来实现的. 当你在缓存函数外部修改缓存对象, 你也在无意中修改了缓存中的对象.这就意味着你之后的每一次调用缓存函数, 返回的都是错误的值.

Since this behavior is usually not what you'd expect, Streamlit tries to be helpful and show you a warning, along with some ideas about how to fix your code.

In this specific case, the fix is just to not mutate `res["output"]` outside the cached function. There was no good reason for us to do that anyway! Another solution would be to clone the result value with `res = deepcopy(expensive_computation(2, 21))`. Check out the section entitled [Fixing caching issues](https://docs.streamlit.io/knowledge-base/using-streamlit/caching-issues) for more information on these approaches and more.



## Advanced caching

高阶缓存

In [caching](https://docs.streamlit.io/library/advanced-features/caching), you learned about the Streamlit cache, which is accessed with the [`@st.cache`](https://docs.streamlit.io/library/api-reference/performance/st.cache) decorator. In this article you'll see how Streamlit's caching functionality is implemented, so that you can use it to improve the performance of your Streamlit apps.

The cache is a key-value store, where the key is a hash of:

缓存是以键值的形式存在的, 键是使用下面提及内容作为哈希值的.

1. The input parameters that you called the function with
2. The value of any external variable used in the function
3. The body of the function
4. The body of any function used inside the cached function

And the value is a tuple of:

这里的值是元组的形式

- The cached output, 缓存的结果
- A hash of the cached output (you'll see why soon), 缓存结果的hash

For both the key and the output hash, Streamlit uses a specialized hash function that knows how to traverse code, hash special objects, and can have its [behavior customized by the user](https://docs.streamlit.io/library/advanced-features/caching#the-hash_funcs-parameter).

对于键和输出哈希, streamlit使用了专门的哈希函数, 这个函数知道如何遍历代码, 哈希指定的对象, 这个函数也可以用户自定义(也就是计算哈希的方式, 用户可以自定义, 即用户希望streamlit通过何种方式实现这个哈希的计算, 间接实现控制被装饰的函数的加载情况).

For example, when the function `expensive_computation(a, b)`, decorated with [`@st.cache`](https://docs.streamlit.io/library/api-reference/performance/st.cache), is executed with `a=2` and `b=21`, Streamlit does the following:

例如, 当被装饰的函数被执行时, streamlit会执行以下操作.

1. Computes the cache key

1. 计算缓存键哈希

2. If the key is found in the cache, then:

   如果键存在在缓存中

   - Extracts the previously-cached (output, output_hash) tuple.

   - 取出之前的缓存值(以元组方式)

   - Performs an Output Mutation Check, where a fresh hash of the output is computed and compared to the stored

      执行一个输出结果可变检测, 输出内容新的哈希值会被计算和已存储的相比较

     (这是因为缓存保持的是引用值, 而可变对象, 引用值不发生改变的前提下, 其内容也可能发生改变, 所以需要执行这一步来校检前后内容的一致)

     - If the two hashes are different, shows a **Cached Object Mutated** warning. (Note: Setting `allow_output_mutation=True` disables this step).
   - 如果哈希存在差异, 将触发可变缓存对象警告(注: 可以取消掉该警告, 使用参数: allow_output_mutation=True)
   
4. If the input key is not found in the cache, then:

   如果输出的键没有发现在缓存中

   - Executes the cached function (i.e. `output = expensive_computation(2, 21)`).
   - 执行缓存函数
   - Calculates the `output_hash` from the function's `output`.
   - 计算输出结果的hash
   - Stores `key → (output, output_hash)` in the cache.
   - 存储键值哈希

5. Returns the output.

4. 返回结果

If an error is encountered an exception is raised. If the error occurs while hashing either the key or the output an `UnhashableTypeError` error is thrown. If you run into any issues, see [fixing caching issues](https://docs.streamlit.io/knowledge-base/using-streamlit/caching-issues).

如果遇到错误, 将触发异常. 如果错误时在键和结果进行哈希计算时触发的, 将会抛出不可哈希类型错误.

### The hash_funcs parameter

hash函数的参数

注: 这里实际上涉及的是, 自定义哈希函数

As described above, Streamlit's caching functionality relies on hashing to calculate the key for cached objects, and to detect unexpected mutations in the cached result.

如上所述, streamlit缓存功能是基于计算缓存对象的哈希值作为键. 以检测缓存内容是否发生改变

For added expressive power, Streamlit lets you override this hashing process using the `hash_funcs` argument. Suppose you define a type called `FileReference` which points to a file in the filesystem:

为了增强可用性, streamlit允许你重写哈希函数用`hash_funcs`参数. 假如你可以定义一个类型名为文件引用(`FileReference`), 这是指向文件系统中的一个文件.

```python
class FileReference:
    def __init__(self, filename):
        self.filename = filename


@st.cache
def func(file_reference):
    ...
```

By default, Streamlit hashes custom classes like `FileReference` by recursively navigating their structure. In this case, its hash is the hash of the filename property. As long as the file name doesn't change, the hash will remain constant.

默认状态下, streamlit哈希自定义的类, 像 `FileReference`, 通过递归的方式来遍历其结构. 在某些情况下, 这个哈希值可以是文件名属性的哈希值. 只要文件名不发生改变, 哈希值就一直不变.

However, what if you wanted to have the hasher check for changes to the file's modification time, not just its name? This is possible with [`@st.cache`](https://docs.streamlit.io/library/api-reference/performance/st.cache)'s `hash_funcs` parameter:

然后, 如果你想希望这个哈希函数去检查文件修改的时间, 而不仅仅是文件名. 

```python
class FileReference:
    def __init__(self, filename):
        self.filename = filename

def hash_file_reference(file_reference):
    filename = file_reference.filename
    return (filename, os.path.getmtime(filename))

@st.cache(hash_funcs={FileReference: hash_file_reference})
def func(file_reference):
    ...
```

Additionally, you can hash `FileReference` objects by the file's contents:

此外你也可以哈希`FileReference`对象通过文件的内容

```python
class FileReference:
    def __init__(self, filename):
        self.filename = filename

def hash_file_reference(file_reference):
    with open(file_reference.filename) as f:
      return f.read()

@st.cache(hash_funcs={FileReference: hash_file_reference})
def func(file_reference):
    ...
```

#### Note

注意

Because Streamlit's hash function works recursively, you don't have to hash the contents inside `hash_file_reference` Instead, you can return a primitive type, in this case the contents of the file, and Streamlit's internal hasher will compute the actual hash from it.

因为streamlit哈希函数是以递归方式运行的, 

### Typical hash functions

While it's possible to write custom hash functions, let's take a look at some of the tools that Python provides out of the box. Here's a list of some hash functions and when it makes sense to use them.



Python's [`id`](https://docs.python.org/3/library/functions.html#id) function | [Example](https://docs.streamlit.io/library/advanced-features/caching#example-1-pass-a-database-connection-around)

- Speed: Fast
- Use case: If you're hashing a singleton object, like an open database connection or a TensorFlow session. These are objects that will only be instantiated once, no matter how many times your script reruns.
- 使用场景: 如果你正在哈希一个单例对象, 如数据库连接实例, 或者TensorFlow会话. 这些对象只会实例化一次, 不管你的脚本运行多少次.

`lambda _: None` | [Example](https://docs.streamlit.io/library/advanced-features/caching#example-2-turn-off-hashing-for-a-specific-type)

- Speed: Fast
- Use case: If you want to turn off hashing of this type. This is useful if you know the object is not going to change.
- 你希望关闭针对特定类型的哈希功能. 这非常有用如果你知道这个对象不会发生改变.

Python's [`hash()`](https://docs.python.org/3/library/functions.html#hash) function | [Example](https://docs.streamlit.io/library/advanced-features/caching#example-3-use-pythons-hash-function)

- Speed: Can be slow based the size of the object being cached
- 速度, 快慢和缓存对象的大小有关
- Use case: If Python already knows how to hash this type correctly.
- 使用场景, 

Custom hash function | [Example](https://docs.streamlit.io/library/advanced-features/caching#the-hash_funcs-parameter)

- Speed: N/a
- 速度: 未知
- Use case: If you'd like to override how Streamlit hashes a particular type.
- 使用场景:  希望重写streamlit hash函数针对特定的类型.

### Example 1: Pass a database connection around

案例1: 传递数据库连接实例

Suppose we want to open a database connection that can be reused across multiple runs of a Streamlit app. For this you can make use of the fact that cached objects are stored by reference to automatically initialize and reuse the connection:

假设我们想打开一个数据库连接, 可以在streamlit app多次运行中可以重复使用.

```python
@st.cache(allow_output_mutation=True)
def get_database_connection():
    return db.get_connection()
```

With just 3 lines of code, the database connection is created once and stored in the cache. Then, every subsequent time `get_database_conection` is called, the already-created connection object is reused automatically. In other words, it becomes a singleton.

只需要3行代码, 数据库连接就可以一次性创建和缓存. 之后的每次`get_database_conection`调用, 这个早已经创建的连接对象将自动可以复用. 换言之, 这个对象成为了单例.

#### Tip

提示

Use the `allow_output_mutation=True` flag to suppress the immutability check. This prevents Streamlit from trying to hash the output connection, and also turns off Streamlit's mutation warning in the process.

使用`allow_output_mutation=True`标记阻止不变性(`immutability`)检查. 这是防止streamlit尝试去hash这个输出连接, 同时这个标记同样会关闭可变(对象)警告.

What if you want to write a function that receives a database connection as input? For that, you'll use `hash_funcs`:

如果你想写一个函数用于接收一个数据库连接作为输入? 这个时候可以使用`hash_funcs`.

```python
@st.cache(hash_funcs={DBConnection: id})
def get_users(connection):
    # Note: We assume that connection is of type DBConnection.
    return connection.execute_sql('SELECT * from Users')
```

Here, we use Python's built-in `id` function, because the connection object is coming from the Streamlit cache via the `get_database_conection` function. This means that the same connection instance is passed around every time, and therefore it always has the same id. However, if you happened to have a second connection object around that pointed to an entirely different database, it would still be safe to pass it to `get_users` because its id is guaranteed to be different than the first id.

这里我们使用python内置的id函数, 因为这个连接对象来自streamlit缓存, 通过`get_database_conection`. 这意味着每次传递的都是相同的连接实例对象, 因此每次都都是相同的id. 

These design patterns apply any time you have an object that points to an external resource, such as a database connection or Tensorflow session.



### Example 2: Turn off hashing for a specific type

案例2: 对特定的类型关闭hash功能

You can turn off hashing entirely for a particular type by giving it a custom hash function that returns a constant. One reason that you might do this is to avoid hashing large, slow-to-hash objects that you know are not going to change. For example:

你可以整体关闭hash针对特定的类型, 通过一个自定义hash函数返回一个常量.

注: 即不希望对某些传入类型产生缓存

```python
@st.cache(hash_funcs={pd.DataFrame: lambda _: None})
def func(huge_constant_dataframe):
    ...
```

When Streamlit encounters an object of this type, it always converts the object into `None`, no matter which instance of `FooType` its looking at. This means all instances are hash to the same value, which effectively cancels out the hashing mechanism.

当streamlit遇到这种类型的对象(注: 手动标记这种类型), 它通常会将这个对象转为`None`, 不管`FooType`的实例类型是哪种. 这意味着所有的实例得到相同的hash值, 这就相当于取消哈希的机制.

### Example 3: Use Python's hash() function

案例3: 使用python内置hash函数

Sometimes, you might want to use Python’s default hashing instead of Streamlit's. For example, maybe you've encountered a type that Streamlit is unable to hash, but it's hashable with Python's built-in `hash()` function:

有时, 你也许想使用python默认的hash功能来取代streamlit内置的. 例如你也许会遇到streamlit内置hash无法处理的数据类型, 但是可以用python内置的hash来处理的情况.

```python
@st.cache(hash_funcs={FooType: hash})
def func(...):
    ...
```