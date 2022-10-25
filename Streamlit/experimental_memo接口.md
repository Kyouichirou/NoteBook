# experimental_memo接口

## st.experimental_memo

Function decorator to memoize function executions.

Memoized data is stored in "pickled" form, which means that the return value of a memoized function must be pickleable.

缓存的数据被序列化形式存储, 这意味着缓存中返回的值必须可以序列化.

Each caller of a memoized function gets its own copy of the cached data.

每次的对缓存函数的数据请求, 得到的是缓存数据的拷贝值.

You can clear a memoized function's cache with f.clear().

你可以使用`clear`方法清除掉缓存

| Function signature                                           |                                                              |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| st.experimental_memo(func=None, *, persist=None, show_spinner=True, suppress_st_warning=False, max_entries=None, ttl=None) |                                                              |
| Parameters                                                   |                                                              |
| **func** *(callable)*                                        | The function to memoize. Streamlit hashes the function's source code. 需要缓存的函数 |
| **persist** *(str or None)* - 持续化                         | Optional location to persist cached data to. Currently, the only valid value is "disk", which will persist to the local disk. 缓存数据持续化存储. 目前仅支持的参数是'disk'(磁盘), 持续化存储于磁盘. |
| **show_spinner** *(boolean)*                                 | Enable the spinner. Default is True to show a spinner when there is a cache miss. 缓存丢失提示, 默认开启 |
| **suppress_st_warning** *(boolean)*                          | Suppress warnings about calling Streamlit functions from within the cached function.忽视警告在缓存的函数执行时 |
| **max_entries** *(int or None)*                              | The maximum number of entries to keep in the cache, or None for an unbounded cache. (When a new entry is added to a full cache, the oldest cached entry will be removed.) The default is None. 缓存最大缓存的实体的数量, 默认None, 无限存储, 当设置数量, 存储数量满了之后, 最先缓存的数据将被清除掉. |
| **ttl** *(float or None)*                                    | The maximum number of seconds to keep an entry in the cache, or None if cache entries should not expire. The default is None. Note that ttl is incompatible with persist="disk" - ttl will be ignored if persist is specified.最大保存缓存秒数, 默认None, 无限时间, 和可持续化参数冲突. |

#### Example

案例

> ```python
> @st.experimental_memo
> def fetch_and_clean_data(url):
>     # Fetch data from URL here, and then clean it up.
>     return data
> 
> d1 = fetch_and_clean_data(DATA_URL_1)
> # Actually executes the function, since this is the first time it was
> # encountered.
> 
> d2 = fetch_and_clean_data(DATA_URL_1)
> # Does not execute the function. Instead, returns its previously computed
> # value. This means that now the data in d1 is the same as in d2.
> 
> d3 = fetch_and_clean_data(DATA_URL_2)
> # This is a different URL, so the function executes.
> ```
>
> To set the `persist` parameter, use this command as follows:
>
> 使用持续化(`persist`)参数的方法:
>
> ```python
> @st.experimental_memo(persist="disk")
> def fetch_and_clean_data(url):
>     # Fetch data from URL here, and then clean it up.
>     return data
> ```
>
> By default, all parameters to a memoized function must be hashable. Any parameter whose name begins with `_` will not be hashed. You can use this as an "escape hatch" for parameters that are not hashable:
>
> 默认情况下, 函数中可以被缓存的参数必须是可以哈希的. 任意参数的命名, 以下划线开头的是不可哈希的. 可以使用这个标记作为不可哈希的参数的标志.
>
> 注: python的可哈希的对象有
>
> > 简要的说可哈希的数据类型，即不可变的数据结构(**数字类型（int，float，bool）字符串str、元组tuple、自定义类的对象**)。
>
> ```python
> @st.experimental_memo
> def fetch_and_clean_data(_db_connection, num_rows):
>     # Fetch data from _db_connection here, and then clean it up.
>     return data
> 
> connection = make_database_connection()
> d1 = fetch_and_clean_data(connection, num_rows=10)
> # Actually executes the function, since this is the first time it was
> # encountered.
> 
> another_connection = make_database_connection()
> d2 = fetch_and_clean_data(another_connection, num_rows=10)
> # Does not execute the function. Instead, returns its previously computed
> # value - even though the _database_connection parameter was different
> # in both calls.
> ```
>
> A memoized function's cache can be procedurally cleared:
>
> 缓存的清除方法
>
> ```python
> @st.experimental_memo
> def fetch_and_clean_data(_db_connection, num_rows):
>     # Fetch data from _db_connection here, and then clean it up.
>     return data
> 
> fetch_and_clean_data.clear()
> # Clear all cached entries for this function.
> ```

Persistent memo caches currently don't support TTL. `ttl` will be ignored if `persist` is specified:

持续化缓存目前不支持`TTL`, `ttl`参数将被忽视, 如果`persist`函数被使用.

```python
import streamlit as st

@st.experimental_memo(ttl=60, persist="disk")
def load_data():
    return 42

st.write(load_data())
```

And a warning will be logged to your terminal:

一个警告将出现在终端上.

```bash
$ streamlit run app.py

  You can now view your Streamlit app in your browser.
  Local URL: http://localhost:8501
  Network URL: http://192.168.1.1:8501

2022-09-22 13:35:41.587 The memoized function 'load_data' has a TTL that will be ignored. Persistent memo caches currently don't support TTL.
```

### Replay static `st` elements in cache-decorated functions

缓存函数中的重复显示静态`st`创建的元素

Functions decorated with `@st.experimental_memo` can contain static `st` elements. When a cache-decorated function is executed, we record the element and block messages produced, so the elements will appear in the app even when execution of the function is skipped because the result was cached.

`@st.experimental_memo`装饰的函数内部可以包含`st`创建元素的方法. 当缓存装饰的函数执行时, 元素创建会被记录下来, 以及拦截信息产生. 所以相关的元素还是会出现, 当该函数因为数据已缓存而跳过执行.

In the example below, the `@st.experimental_memo` decorator is used to cache the execution of the `load_data` function, that returns a pandas DataFrame. Notice the cached function also contains a `st.area_chart` command, which will be replayed when the function is skipped because the result was cached.

在下面的案例, `@st.experimental_memo`装饰器被用在`load_data`函数上, 返回pandas的dataframe. 注意这个函数包含`st.area_chart`命令, 这个方法不会执行, 因为数据已经缓存.

```python
import numpy as np
import pandas as pd
import streamlit as st

@st.experimental_memo
def load_data(rows):
    chart_data = pd.DataFrame(
        np.random.randn(rows, 10),
        columns=["a", "b", "c", "d", "e", "f", "g", "h", "i", "j"],
    )
    # Contains a static element st.area_chart
    # This will be recorded and displayed even when the function is skipped
    # 这将被即可和展示, 就算这个函数跳过执行
    st.area_chart(chart_data)
    # 绘制图表, 在缓存内部
    # 注意
    return chart_data

df = load_data(20)
# dataframe数据展示
st.dataframe(df)
```

*上述含义, 即, 尽管添加了缓存装饰器, 在第二次执行时, 尽管这个函数不会执行直接返回执行缓存的结果, 但是函数内部包含的绘制图形也会被展示出来.*

![img](https://docs.streamlit.io/images/replay-cached-elements.png)Supported static `st` elements in cache-decorated functions include:

支持静态`st`元素在缓存装饰器装饰的函数

- `st.alert`
- `st.altair_chart`
- `st.area_chart`
- `st.bar_chart`
- `st.ballons`
- `st.bokeh_chart`
- `st.caption`
- `st.code`
- `st.components.v1.html`
- `st.components.v1.iframe`
- `st.container`
- `st.dataframe`
- `st.echo`
- `st.empty`
- `st.error`
- `st.exception`
- `st.expander`
- `st.experimental_get_query_params`
- `st.experimental_set_query_params`
- `st.graphviz_chart`
- `st.help`
- `st.info`
- `st.json`
- `st.latex`
- `st.line_chart`
- `st.markdown`
- `st.metric`
- `st.plotly_chart`
- `st.progress`
- `st.pydeck_chart`
- `st.snow`
- `st.spinner`
- `st.success`
- `st.table`
- `st.text`
- `st.vega_lite_chart`
- `st.warning`

All other `st` commands, including widgets, forms, and media elements are not supported in cache-decorated functions. If you use them, the code will only be called when we detect a cache "miss", which can lead to unexpected results. Which is why Streamlit will throw a `CachedStFunctionWarning`, like the one below:

除上述的函数, 其他的所有的函数均不支持缓存装饰器函数, 如widget, forms, 多媒体元素等. 如果使用不支持的函数, 代码在执行时会检测到缓存丢失(`miss`)的警告, 这会导致不如预期的结果. 这也是为什么streamlit抛出`CachedStFunctionWarning`警告的原因.

```python
import numpy as np
import pandas as pd
import streamlit as st

@st.experimental_memo
def load_data(rows):
    chart_data = pd.DataFrame(
        np.random.randn(rows, 10),
        columns=["a", "b", "c", "d", "e", "f", "g", "h", "i", "j"],
    )
    # Contains an unsupported st command
    # 不支持的函数
    # Streamlit will throw a CachedStFunctionWarning
    # 将抛出警告
    st.slider("Select a value", 0, 10, 5) 

    return chart_data

df = load_data(20)
st.dataframe(df)
```

![img](https://docs.streamlit.io/images/cached-st-function-warning.png)