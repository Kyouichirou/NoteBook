# streamlit进阶-缓存/单例/会话状态

相关文档参考, [performance](https://docs.streamlit.io/library/api-reference/performance)

使用streamlit前需要了解一个关键信息, streamlit代码的执行逻辑.

> Streamlit's unique execution model is a part of what makes it a joy to use: your code executes from top to bottom like a simple script for every interaction. There's no need to think about models, views, controllers, or anything of the sort.
>
> streamlit的独特的执行模式让它在使用时看起来非常搞笑, 每次交互, 代码会像一个普通的脚本一样被从头到尾执行一遍.
>
> -- 来自官方文档的自我调侃

*注: 这种执行机制将会引发很多问题, 如数据需要反复被加载, 假如每次加载都需要大量时间, 这将会是个大问题; 以及对于使用数据库连接, 不可能每次运行都连接一次数据库. 所以streamlit提供了多种的缓存机制用于解决这个问题.*

```python
import streamlit as st
import time

@st.cache(suppress_st_warning=True)
def expensive_computation(a, b):
    st.write("Cache miss: expensive_computation(", a, ",", b, ") ran")
    # This makes the function take 2s to run
    time.sleep(2)
    return a * b

a = 2
# 👈 Changed this
b = st.slider("Pick a number", 0, 10)
# 注意代码的执行逻辑和直接的代码感官-调整数值(新的交互触发页面的重新加载), 这个变量改变
res = expensive_computation(a, b)
st.write("Result:", res)
```

这个示例, 可以作为演示这种执行逻辑, 当变更slider(滑动条), 修改参数时, 相关的代码就会重新执行一遍. (因为变更输入的参数, 会引发缓存丢失的警告)

涉及到以下四个API接口

- st.cache
- st.session_state
- st.experimental_memo
- st.experimental_singleton

`st.cache`, 是目前实现缓存的主要方式, 但是官方目前准备使用`st.experimental_memo`, `st.experimental_singleton`这两个接口用于取代`st.cache`.

> These specialized **memoization** and **singleton** commands represent a big step in Streamlit's evolution, with the potential to *entirely replace* `@st.cache` at some point in 2022.
>
> 这些针对性的记忆(memoization)和单例(singleton)命令代表着streamlit进化的一大步, 将在2022年可能完全替代@st.cache装饰器在某些方面.

目前两个接口依然还是处于试验的状态, 官方尚未完成这个替换.

`st.experimental_memo`这个接口负责缓存函数生成返回的结果

`st.experimental_singleton`这个接口负责单例对象的缓存(非数据型对象), 这个需要注意的是, 相关的缓存对象的**线程安全**, 因为涉及到多线程同时访问同一对象, 如: 数据库连接的实例.

这两个功能在`st.cache`上是不做区分的, 新的接口通过不明的名称区分开在不同场景下的使用.

相比于`st.cache`, 新的缓存机制, 对不同的缓存需要做了区分, 代码逻辑上更为清晰; 同时其底层的实现也有别于`st.cache`, 其运行速度更快, 官方宣称在某些场景的使用, 可以快上1个量级.

`st.cache`支持的函数更多, 如自定义哈希函数.

`st.session_state`, 实现多个用户同时访问app, 分享全局会话变量. 同时对于不是线程安全的单例模式的对象, 不适合使用`st.experimental_singleton`装饰器, 可以将对象临时保存到`st.session_state`下. 这个接口相当于对上述的缓存/单例的补充, 亦或者是新的缓存机制尚未成熟前, 使用这个来过度?

## 缓存-全局

使用上面使用的代码作为测试.

st.cache`作用于全局, 需要注意这个问题的存在, 假如多用户访问.

![global](https://p0.meituan.net/dpplatform/638690416319a048b6bc072c446f3df854033.png)



## 缓存的其他使用

如在数据库检索中, 为了减少非必要的检索, 可以设置缓存时间, 每隔一段时间才执行新的检索.

[`st.secrets`](https://docs.streamlit.io/streamlit-cloud/get-started/deploy-an-app/connect-to-data-sources/secrets-management)

> It's generally considered bad practice to store unencrypted secrets in a git repository. If your application needs access to sensitive credentials the recommended solution is to store those credentials in a file that is not committed to the repository and to pass them as environment variables.
>
> Secrets Management allows you to store secrets securely and access them in your Streamlit app as environment variables.

```python
# 连接MongoDB
import streamlit as st
import pymongo

# 初始化连接
@st.experimental_singleton
def init_connection():
    # 在连接MongoDB时, 是不需要用户名和密码的
    # client = MongoClient('localhost', 27017)
    # client = MongoClient('mongodb://localhost:27017/')
    # 以上两种方式等价的
    return pymongo.MongoClient(**st.secrets["mongo"])

client = init_connection()

# 在检索数据时, 进行时间范围内的缓存, ttl
@st.experimental_memo(ttl=600)
def get_data():
    db = client.mydb
    items = db.mycollection.find()
    items = list(items)
    return items

items = get_data()

for item in items:
    st.write(f"{item['name']} has a :{item['pet']}:")
```

MongoDB简单使用使用

```python
from pymongo import MongoClient

client = MongoClient()
# 类似于js的访问对象, 类似
db = client.test_database
# 或者这样
db = client['test-database']
```

MongoDB中的库, 表, 行

| MySQL       | MongoDB     | 含义                                                |
| ----------- | ----------- | --------------------------------------------------- |
| table       | collection  | 数据库表/集合                                       |
| row         | document    | 数据记录行/文档                                     |
| column      | field       | 数据字段/域                                         |
| index       | index       | 索引                                                |
| table joins |             | 表连接, MongoDB不支持(应该是勉强也支持一些并表功能) |
| primary key | primary key | 主键,MongoDB自动将_id字段设置为主键                 |
