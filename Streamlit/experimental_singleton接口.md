# experimental_singleton接口

## st.experimental_singleton

Function decorator to store singleton objects.

Each singleton object is shared across all users connected to the app. Singleton objects *must* be thread-safe, because they can be accessed from multiple threads concurrently.

每个单例对象都可以被所有当前app的用户所共享. 单例对象必须是线程安全, 因为这个对象可能同时被多个线程访问.

(If thread-safety is an issue, consider using `st.session_state` to store per-session singleton objects instead.)

如果线程安全是个问题, 可以考虑使用`st.session_state`取代单例对象.

> 在并发编程时，如果多个线程访问同一资源，我们需要保证访问的时候不会产生冲突，数据修改不会发生错误，这就是我们常说的 线程安全 。

You can clear a memoized function's cache with f.clear().

`clear`方法可以清除掉缓存.

| Function signature                                           |                                                              |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| st.experimental_singleton(func=None, *, show_spinner=True, suppress_st_warning=False) |                                                              |
| Parameters                                                   |                                                              |
| **func** *(callable)*                                        | The function that creates the singleton. Streamlit hashes the function's source code. |
| **show_spinner** *(boolean)*                                 | Enable the spinner. Default is True to show a spinner when there is a "cache miss" and the singleton is being created. 默认参数为TRUE, 当出现缓存丢失提示, 当单例对象创建时 |
| **suppress_st_warning** *(boolean)*                          | Suppress warnings about calling Streamlit functions from within the singleton function. 阻止来自单例函数的警告. |

#### Example

案例

> ```python
> @st.experimental_singleton
> def get_database_session(url):
>     # Create a database session object that points to the URL.
>     return session
> 
> s1 = get_database_session(SESSION_URL_1)
> # Actually executes the function, since this is the first time it was
> # encountered.
> 
> s2 = get_database_session(SESSION_URL_1)
> # Does not execute the function. Instead, returns its previously computed
> # value. This means that now the connection object in s1 is the same as in s2.
> 
> s3 = get_database_session(SESSION_URL_2)
> # This is a different URL, so the function executes.
> ```
>
> By default, all parameters to a singleton function must be hashable. Any parameter whose name begins with `_` will not be hashed. You can use this as an "escape hatch" for parameters that are not hashable:
>
> 默认情况下, 函数中可以被缓存的参数必须是可以哈希的. 任意参数的命名, 以下划线开头的是不可哈希的. 可以使用这个标记作为不可哈希的参数的标志.
>
> ```python
> @st.experimental_singleton
> def get_database_session(_sessionmaker, url):
>     # Create a database connection object that points to the URL.
>     return connection
> 
> s1 = get_database_session(create_sessionmaker(), DATA_URL_1)
> # Actually executes the function, since this is the first time it was
> # encountered.
> 
> s2 = get_database_session(create_sessionmaker(), DATA_URL_1)
> # Does not execute the function. Instead, returns its previously computed
> # value - even though the _sessionmaker parameter was different
> # in both calls.
> ```
>
> A singleton function's cache can be procedurally cleared:
>
> 缓存的清除方法
>
> ```python
> @st.experimental_singleton
> def get_database_session(_sessionmaker, url):
>     # Create a database connection object that points to the URL.
>     return connection
> 
> get_database_session.clear()
> # Clear all cached entries for this function.
> ```



### Replay static `st` elements in cache-decorated functions

缓存函数中的重复显示静态`st`创建的元素

Functions decorated with `@st.experimental_singleton` can contain static `st` elements. When a cache-decorated function is executed, we record the element and block messages produced, so the elements will appear in the app even when execution of the function is skipped because the result was cached.

`@st.experimental_singleton`装饰的函数内部可以包含`st`创建元素的方法. 当缓存装饰的函数执行时, 元素创建会被记录下来, 以及拦截信息产生. 所以相关的元素还是会出现, 当该函数因为数据已缓存而跳过执行.

In the example below, the `@st.experimental_singleton` decorator is used to cache the execution of the `get_model` function, that returns a 🤗 Hugging Face Transformers model. Notice the cached function also contains a `st.bar_chart` command, which will be replayed when the function is skipped because the result was cached.

在下面的案例, `@st.experimental_singleton`装饰器被用在`get_model`函数上, 返回pandas的dataframe. 注意这个函数包含`st.bar_chart`命令, 这个方法不会执行, 因为数据已经缓存.

```python
import numpy as np
import pandas as pd
import streamlit as st
from transformers import AutoModel

@st.experimental_singleton
def get_model(model_type):
    # Contains a static element st.bar_chart
    st.bar_chart(
        np.random.rand(10, 1)
    )  # This will be recorded and displayed even when the function is skipped

    # Create a model of the specified type
    return AutoModel.from_pretrained(model_type)

bert_model = get_model("distilbert-base-uncased")
st.help(bert_model) # Display the model's docstring
```

![img](https://docs.streamlit.io/images/replay-cached-elements-singleton.png)

Supported static `st` elements in cache-decorated functions include:

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
from transformers import AutoModel

@st.experimental_singleton
def get_model(model_type):
    # Contains an unsupported st command
    st.slider("Select a value", 0, 10, 5) # Streamlit will throw a CachedStFunctionWarning

    # Create a model of the specified type
    return AutoModel.from_pretrained(model_type)

bert_model = get_model("distilbert-base-uncased")
st.help(bert_model) # Display the model's docstring
```

![img](https://docs.streamlit.io/images/cached-st-function-warning-singleton.png)