# 查询参数和字符串校验

**FastAPI** 允许你为参数声明额外的信息和校验。

让我们以下面的应用程序为例：

=== "Python 3.10+"

    ```Python hl_lines="7"
    {!> ../../../docs_src/query_params_str_validations/tutorial001_py310.py!}
    ```

=== "Python 3.6+"

    ```Python hl_lines="9"
    {!> ../../../docs_src/query_params_str_validations/tutorial001.py!}
    ```

查询参数 `q` 的类型为 `Union[str, None]`（在 Python 3.10 中则是 `str | None`），that means that it's of type `str` but could also be `None`, and indeed, the default value is `None`, so FastAPI will know it's not required.

!!! note
    FastAPI will know that the value of `q` is not required because of the default value `= None`.

    The `Union` in `Union[str, None]` will allow your editor to give you better support and detect errors.

## 额外的校验

我们打算添加约束条件：即使 `q` 是可选的，但只要提供了该参数，则**该参数值的长度不能超过50个字符**。

### 导入 `Query` 与 `Annotated`

为此，首先需要导入：

* `Query` from `fastapi`
* `Annotated` from `typing` (or from `typing_extensions` in Python below 3.9)

=== "Python 3.10+"

    In Python 3.9 or above, `Annotated` is part of the standard library, so you can import it from `typing`.

    ```Python hl_lines="1  3"
    {!> ../../../docs_src/query_params_str_validations/tutorial002_an_py310.py!}
    ```

=== "Python 3.6+"

    In versions of Python below Python 3.9 you import `Annotated` from `typing_extensions`.

    It will already be installed with FastAPI.

    ```Python hl_lines="3-4"
    {!> ../../../docs_src/query_params_str_validations/tutorial002_an.py!}
    ```

!!! info
    FastAPI added support for `Annotated` (and started recommending it) in version 0.95.0.

    If you have an older version, you would get errors when trying to use `Annotated`.

    Make sure you [Upgrade the FastAPI version](../deployment/versions.md#upgrading-the-fastapi-versions){.internal-link target=_blank} to at least 0.95.1 before using `Annotated`.

## Use `Annotated` in the type for the `q` parameter

Remember I told you before that `Annotated` can be used to add metadata to your parameters in the [Python Types Intro](../python-types.md#type-hints-with-metadata-annotations){.internal-link target=_blank}?

Now it's the time to use it with FastAPI. 🚀

We had this type annotation:

=== "Python 3.10+"

    ```Python
    q: str | None = None
    ```

=== "Python 3.6+"

    ```Python
    q: Union[str, None] = None
    ```

What we will do is wrap that with `Annotated`, so it becomes:

=== "Python 3.10+"

    ```Python
    q: Annotated[str | None] = None
    ```

=== "Python 3.6+"

    ```Python
    q: Annotated[Union[str, None]] = None
    ```

Both of those versions mean the same thing, `q` is a parameter that can be a `str` or `None`, and by default, it is `None`.

Now let's jump to the fun stuff. 🎉

## Add `Query` to `Annotated` in the `q` parameter

Now that we have this `Annotated` where we can put more metadata, add `Query` to it, and set the parameter `max_length` to 50:

=== "Python 3.10+"

    ```Python hl_lines="9"
    {!> ../../../docs_src/query_params_str_validations/tutorial002_an_py310.py!}
    ```

=== "Python 3.6+"

    ```Python hl_lines="10"
    {!> ../../../docs_src/query_params_str_validations/tutorial002_an.py!}
    ```

Notice that the default value is still `None`, so the parameter is still optional.

But now, having `Query(max_length=50)` inside of `Annotated`, we are telling FastAPI that we want it to extract this value from the query parameters (this would have been the default anyway 🤷) and that we want to have **additional validation** for this value (that's why we do this, to get the additional validation). 😎

FastAPI will now:

* **Validate** the data making sure that the max length is 50 characters
* Show a **clear error** for the client when the data is not valid
* **Document** the parameter in the OpenAPI schema *path operation* (so it will show up in the **automatic docs UI**)

## Alternative (old) `Query` as the default value

Previous versions of FastAPI (before <abbr title="before 2023-03">0.95.0</abbr>) required you to use `Query` as the default value of your parameter, instead of putting it in `Annotated`, there's a high chance that you will see code using it around, so I'll explain it to you.

!!! tip
    For new code and whenever possible, use `Annotated` as explained above. There are multiple advantages (explained below) and no disadvantages. 🍰

This is how you would use `Query()` as the default value of your function parameter, setting the parameter `max_length` to 50:

=== "Python 3.10+"

    ```Python hl_lines="7"
    {!> ../../../docs_src/query_params_str_validations/tutorial002_py310.py!}
    ```

=== "Python 3.6+"

    ```Python hl_lines="9"
    {!> ../../../docs_src/query_params_str_validations/tutorial002.py!}
    ```

As in this case (without using `Annotated`) we have to replace the default value `None` in the function with `Query()`, we now need to set the default value with the parameter `Query(default=None)`, it serves the same purpose of defining that default value (at least for FastAPI).

所以：

```Python
q: Union[str, None] = Query(default=None)
```

...使得参数可选，with a default value of `None`，等同于：

```Python
q: Union[str, None] = None
```

And in Python 3.10 and above:

```Python
q: str | None = Query(default=None)
```

...makes the parameter optional, with a default value of `None`, the same as:

```Python
q: str | None = None
```

但是 `Query` 显式地将其声明为查询参数。

!!! info
    Have in mind that the most important part to make a parameter optional is the part:

    ```Python
    = None
    ```

    or the:

    ```Python
    = Query(default=None)
    ```

    as it will use that `None` as the default value, and that way make the parameter **not required**.

    The `Union[str, None]` part allows your editor to provide better support, but it is not what tells FastAPI that this parameter is not required.

然后，我们可以将更多的参数传递给 `Query`。在本例中，适用于字符串的 `max_length` 参数：

```Python
q: Union[str, None] = Query(default=None, max_length=50)
```

将会校验数据，在数据无效时展示清晰的错误信息，并在 OpenAPI 模式的*路径操作*中记录该参​​数。

### `Query` as the default value or in `Annotated`

Have in mind that when using `Query` inside of `Annotated` you cannot use the `default` parameter for `Query`.

Instead use the actual default value of the function parameter. Otherwise, it would be inconsistent.

For example, this is not allowed:

```Python
q: Annotated[str, Query(default="rick")] = "morty"
```

...because it's not clear if the default value should be `"rick"` or `"morty"`.

So, you would use (preferably):

```Python
q: Annotated[str, Query()] = "rick"
```

...or in older code bases you will find:

```Python
q: str = Query(default="rick")
```

### Advantages of `Annotated`

**Using `Annotated` is recommended** instead of the default value in function parameters, it is **better** for multiple reasons. 🤓

The **default** value of the **function parameter** is the **actual default** value, that's more intuitive with Python in general. 😌

You could **call** that same function in **other places** without FastAPI, and it would **work as expected**. If there's a **required** parameter (without a default value), your **editor** will let you know with an error, **Python** will also complain if you run it without passing the required parameter.

When you don't use `Annotated` and instead use the **(old) default value style**, if you call that function without FastAPI in **other place**, you have to **remember** to pass the arguments to the function for it to work correctly, otherwise the values will be different from what you expect (e.g. `QueryInfo` or something similar instead of `str`). And your editor won't complain, and Python won't complain running that function, only when the operations inside error out.

Because `Annotated` can have more than one metadata annotation, you could now even use the same function with other tools, like <a href="https://typer.tiangolo.com/" class="external-link" target="_blank">Typer</a>. 🚀

## 添加更多校验

你还可以添加 `min_length` 参数：

=== "Python 3.10+"

    ```Python hl_lines="10"
    {!> ../../../docs_src/query_params_str_validations/tutorial003_an_py310.py!}
    ```

=== "Python 3.9+"

    ```Python hl_lines="10"
    {!> ../../../docs_src/query_params_str_validations/tutorial003_an_py39.py!}
    ```

=== "Python 3.6+"

    ```Python hl_lines="11"
    {!> ../../../docs_src/query_params_str_validations/tutorial003_an.py!}
    ```

=== "Python 3.10+ non-Annotated"

    !!! tip
        Prefer to use the `Annotated` version if possible.

    ```Python hl_lines="7"
    {!> ../../../docs_src/query_params_str_validations/tutorial003_py310.py!}
    ```

=== "Python 3.6+ non-Annotated"

    !!! tip
        Prefer to use the `Annotated` version if possible.

    ```Python hl_lines="10"
    {!> ../../../docs_src/query_params_str_validations/tutorial003.py!}
    ```

## 添加正则表达式

你可以定义一个参数值必须匹配的<abbr title="正则表达式或正则是定义字符串搜索模式的字符序列。">正则表达式</abbr> `pattern`：

=== "Python 3.9+"

    ```Python hl_lines="11"
    {!> ../../../docs_src/query_params_str_validations/tutorial004_an_py39.py!}
    ```

=== "Python 3.6+"

    ```Python hl_lines="12"
    {!> ../../../docs_src/query_params_str_validations/tutorial004_an.py!}
    ```

=== "Python 3.10+ non-Annotated"

    !!! tip
        Prefer to use the `Annotated` version if possible.

    ```Python hl_lines="9"
    {!> ../../../docs_src/query_params_str_validations/tutorial004_py310.py!}
    ```

=== "Python 3.6+ non-Annotated"

    !!! tip
        Prefer to use the `Annotated` version if possible.

    ```Python hl_lines="11"
    {!> ../../../docs_src/query_params_str_validations/tutorial004.py!}
    ```

这个指定的正则表达式通过以下规则检查接收到的参数值：

* `^`：以该符号之后的字符开头，符号之前没有字符。
* `fixedquery`: 值精确地等于 `fixedquery`。
* `$`: 到此结束，在 `fixedquery` 之后没有更多字符。

如果你对所有的这些**「正则表达式」**概念感到迷茫，请不要担心。对于许多人来说这都是一个困难的主题。你仍然可以在无需正则表达式的情况下做很多事情。

但是，一旦你需要用到并去学习它们时，请了解你已经可以在 **FastAPI** 中直接使用它们。

### Pydantic v1 `regex` instead of `pattern`

Before Pydantic version 2 and before FastAPI 0.100.0, the parameter was called `regex` instead of `pattern`, but it's now deprecated.

You could still see some code using it:

=== "Python 3.10+ Pydantic v1"

    ```Python hl_lines="11"
    {!> ../../../docs_src/query_params_str_validations/tutorial004_an_py310_regex.py!}
    ```

But know that this is deprecated and it should be updated to use the new parameter `pattern`. 🤓

## 默认值

You can, of course, use default values other than `None`.

假设你想要声明查询参数 `q`，使其 `min_length` 为 `3`，并且默认值为 `fixedquery`：

=== "Python 3.9+"

    ```Python hl_lines="9"
    {!> ../../../docs_src/query_params_str_validations/tutorial005_an_py39.py!}
    ```

=== "Python 3.6+"

    ```Python hl_lines="8"
    {!> ../../../docs_src/query_params_str_validations/tutorial005_an.py!}
    ```

=== "Python 3.6+ non-Annotated"

    !!! tip
        Prefer to use the `Annotated` version if possible.

    ```Python hl_lines="7"
    {!> ../../../docs_src/query_params_str_validations/tutorial005.py!}
    ```

!!! note
    具有默认值 of any type, including `None`，还会使该参数成为可选参数。

## 声明为必需参数

当我们不需要声明额外的校验或元数据时，只需不声明默认值就可以使 `q` 参数成为必需参数，例如：

```Python
q: str
```

代替：

```Python
q: Union[str, None] = None
```

但是现在我们正在用 `Query` 声明它，例如：


=== "Annotated"

    ```Python
    q: Annotated[Union[str, None], Query(min_length=3)] = None
    ```

=== "non-Annotated"

    ```Python
    q: Union[str, None] = Query(default=None, min_length=3)
    ```

因此，当你在使用 `Query` 且需要声明一个值是必需的时，只需不声明默认参数：

=== "Python 3.9+"

    ```Python hl_lines="9"
    {!> ../../../docs_src/query_params_str_validations/tutorial006_an_py39.py!}
    ```

=== "Python 3.6+"

    ```Python hl_lines="8"
    {!> ../../../docs_src/query_params_str_validations/tutorial006_an.py!}
    ```

=== "Python 3.6+ non-Annotated"

    !!! tip
        Prefer to use the `Annotated` version if possible.

    ```Python hl_lines="7"
    {!> ../../../docs_src/query_params_str_validations/tutorial006.py!}
    ```

    !!! tip
        Notice that, even though in this case the `Query()` is used as the function parameter default value, we don't pass the `default=None` to `Query()`.

        Still, probably better to use the `Annotated` version. 😉

### 使用省略号（`...`）声明必需参数

有另一种方法可以显式的声明一个值是必需的，即将默认参数的默认值设为 `...`：

=== "Python 3.9+"

    ```Python hl_lines="9"
    {!> ../../../docs_src/query_params_str_validations/tutorial006b_an_py39.py!}
    ```

=== "Python 3.6+"

    ```Python hl_lines="8"
    {!> ../../../docs_src/query_params_str_validations/tutorial006b_an.py!}
    ```

=== "Python 3.6+ non-Annotated"

    !!! tip
        Prefer to use the `Annotated` version if possible.

    ```Python hl_lines="7"
    {!> ../../../docs_src/query_params_str_validations/tutorial006b.py!}
    ```

!!! info
    如果你之前没见过 `...` 这种用法：它是一个特殊的单独值，它是 <a href="https://docs.python.org/3/library/constants.html#Ellipsis" class="external-link" target="_blank">Python 的一部分并且被称为「省略号」</a>。

    Pydantic 和 FastAPI 使用它来显式的声明需要一个值。

这将使 **FastAPI** 知道此查询参数是必需的。

### 使用 `None` 声明必需参数

你可以声明一个参数可以接受 `None` 值，但它仍然是必需的。这将强制客户端发送一个值，即使该值是 `None`。

为此，你可以声明 `None` 是一个有效的类型，并仍然使用 `...` 作为默认值：

=== "Python 3.10+"

    ```Python hl_lines="9"
    {!> ../../../docs_src/query_params_str_validations/tutorial006c_an_py310.py!}
    ```

=== "Python 3.9+"

    ```Python hl_lines="9"
    {!> ../../../docs_src/query_params_str_validations/tutorial006c_an_py39.py!}
    ```

=== "Python 3.6+"

    ```Python hl_lines="10"
    {!> ../../../docs_src/query_params_str_validations/tutorial006c_an.py!}
    ```

=== "Python 3.10+ non-Annotated"

    !!! tip
        Prefer to use the `Annotated` version if possible.

    ```Python hl_lines="7"
    {!> ../../../docs_src/query_params_str_validations/tutorial006c_py310.py!}
    ```

=== "Python 3.6+ non-Annotated"

    !!! tip
        Prefer to use the `Annotated` version if possible.

    ```Python hl_lines="9"
    {!> ../../../docs_src/query_params_str_validations/tutorial006c.py!}
    ```

!!! tip
    Pydantic 是 FastAPI 中所有数据验证和序列化的核心，当你在没有设默认值的情况下使用 `Optional` 或 `Union[Something, None]` 时，Pydantic 具有特殊行为，你可以在 Pydantic 文档中阅读有关<a href="https://pydantic-docs.helpmanual.io/usage/models/#required-optional-fields" class="external-link" target="_blank">必需的可空字段（Required Optional fields）（英文）</a>的更多信息。

### 使用Pydantic中的`Required`代替省略号(`...`)

如果你觉得使用 `...` 不舒服，你也可以从 Pydantic 导入并使用 `Required`：

=== "Python 3.9+"

    ```Python hl_lines="4  10"
    {!> ../../../docs_src/query_params_str_validations/tutorial006d_an_py39.py!}
    ```

=== "Python 3.6+"

    ```Python hl_lines="2  9"
    {!> ../../../docs_src/query_params_str_validations/tutorial006d_an.py!}
    ```

=== "Python 3.6+ non-Annotated"

    !!! tip
        Prefer to use the `Annotated` version if possible.

    ```Python hl_lines="2  8"
    {!> ../../../docs_src/query_params_str_validations/tutorial006d.py!}
    ```

!!! tip
    请记住，在大多数情况下，当你需要某些东西时，可以简单地省略默认值，因此你通常不必使用 `...` 或 `Required`。

## 查询参数列表 / 多个值

当你使用 `Query` 显式地定义查询参数时，你还可以声明它去接收一组值，或换句话来说，接收多个值。

例如，要声明一个可在 URL 中出现多次的查询参数 `q`，你可以这样写：

=== "Python 3.10+"

    ```Python hl_lines="9"
    {!> ../../../docs_src/query_params_str_validations/tutorial011_an_py310.py!}
    ```

=== "Python 3.9+"

    ```Python hl_lines="9"
    {!> ../../../docs_src/query_params_str_validations/tutorial011_an_py39.py!}
    ```

=== "Python 3.6+"

    ```Python hl_lines="10"
    {!> ../../../docs_src/query_params_str_validations/tutorial011_an.py!}
    ```

=== "Python 3.10+ non-Annotated"

    !!! tip
        Prefer to use the `Annotated` version if possible.

    ```Python hl_lines="7"
    {!> ../../../docs_src/query_params_str_validations/tutorial011_py310.py!}
    ```

=== "Python 3.9+ non-Annotated"

    !!! tip
        Prefer to use the `Annotated` version if possible.

    ```Python hl_lines="9"
    {!> ../../../docs_src/query_params_str_validations/tutorial011_py39.py!}
    ```

=== "Python 3.6+ non-Annotated"

    !!! tip
        Prefer to use the `Annotated` version if possible.

    ```Python hl_lines="9"
    {!> ../../../docs_src/query_params_str_validations/tutorial011.py!}
    ```

然后，输入如下网址：

```
http://localhost:8000/items/?q=foo&q=bar
```

你会在*路径操作函数*的*函数参数* `q` 中以一个 Python `list` 的形式接收到*查询参数* `q` 的多个值（`foo` 和 `bar`）。

因此，该 URL 的响应将会是：

```JSON
{
  "q": [
    "foo",
    "bar"
  ]
}
```

!!! tip
    要声明类型为 `list` 的查询参数，如上例所示，你需要显式地使用 `Query`，否则该参数将被解释为请求体。

交互式 API 文档将会相应地进行更新，以允许使用多个值：

<img src="/img/tutorial/query-params-str-validations/image02.png">

### 具有默认值的查询参数列表 / 多个值

你还可以定义在没有任何给定值时的默认 `list` 值：

=== "Python 3.9+"

    ```Python hl_lines="9"
    {!> ../../../docs_src/query_params_str_validations/tutorial012_an_py39.py!}
    ```

=== "Python 3.6+"

    ```Python hl_lines="10"
    {!> ../../../docs_src/query_params_str_validations/tutorial012_an.py!}
    ```

=== "Python 3.9+ non-Annotated"

    !!! tip
        Prefer to use the `Annotated` version if possible.

    ```Python hl_lines="7"
    {!> ../../../docs_src/query_params_str_validations/tutorial012_py39.py!}
    ```

=== "Python 3.6+ non-Annotated"

    !!! tip
        Prefer to use the `Annotated` version if possible.

    ```Python hl_lines="9"
    {!> ../../../docs_src/query_params_str_validations/tutorial012.py!}
    ```

如果你访问：

```
http://localhost:8000/items/
```

`q` 的默认值将为：`["foo", "bar"]`，你的响应会是：

```JSON
{
  "q": [
    "foo",
    "bar"
  ]
}
```

#### 使用 `list`

你也可以直接使用 `list` 代替 `List [str]`（在 Python 3.9 以上的版本则是 `list[str]`）：

=== "Python 3.9+"

    ```Python hl_lines="9"
    {!> ../../../docs_src/query_params_str_validations/tutorial013_an_py39.py!}
    ```

=== "Python 3.6+"

    ```Python hl_lines="8"
    {!> ../../../docs_src/query_params_str_validations/tutorial013_an.py!}
    ```

=== "Python 3.6+ non-Annotated"

    !!! tip
        Prefer to use the `Annotated` version if possible.

    ```Python hl_lines="7"
    {!> ../../../docs_src/query_params_str_validations/tutorial013.py!}
    ```

!!! note
    请记住，在这种情况下 FastAPI 将不会检查列表的内容。

    例如，`List[int]` 将检查（并记录到文档）列表的内容必须是整数。但是单独的 `list` 不会。

## 声明更多元数据

你可以添加更多有关该参数的信息。

这些信息将包含在生成的 OpenAPI 模式中，并由文档用户界面和外部工具所使用。

!!! note
    请记住，不同的工具对 OpenAPI 的支持程度可能不同。

    其中一些可能不会展示所有已声明的额外信息，尽管在大多数情况下，缺少的这部分功能已经计划进行开发。

你可以添加 `title`：

=== "Python 3.10+"

    ```Python hl_lines="10"
    {!> ../../../docs_src/query_params_str_validations/tutorial007_an_py310.py!}
    ```

=== "Python 3.9+"

    ```Python hl_lines="10"
    {!> ../../../docs_src/query_params_str_validations/tutorial007_an_py39.py!}
    ```

=== "Python 3.6+"

    ```Python hl_lines="11"
    {!> ../../../docs_src/query_params_str_validations/tutorial007_an.py!}
    ```

=== "Python 3.10+ non-Annotated"

    !!! tip
        Prefer to use the `Annotated` version if possible.

    ```Python hl_lines="8"
    {!> ../../../docs_src/query_params_str_validations/tutorial007_py310.py!}
    ```

=== "Python 3.6+ non-Annotated"

    !!! tip
        Prefer to use the `Annotated` version if possible.

    ```Python hl_lines="10"
    {!> ../../../docs_src/query_params_str_validations/tutorial007.py!}
    ```

以及 `description`：

=== "Python 3.10+"

    ```Python hl_lines="14"
    {!> ../../../docs_src/query_params_str_validations/tutorial008_an_py310.py!}
    ```

=== "Python 3.9+"

    ```Python hl_lines="14"
    {!> ../../../docs_src/query_params_str_validations/tutorial008_an_py39.py!}
    ```

=== "Python 3.6+"

    ```Python hl_lines="15"
    {!> ../../../docs_src/query_params_str_validations/tutorial008_an.py!}
    ```

=== "Python 3.10+ non-Annotated"

    !!! tip
        Prefer to use the `Annotated` version if possible.

    ```Python hl_lines="12"
    {!> ../../../docs_src/query_params_str_validations/tutorial008_py310.py!}
    ```

=== "Python 3.6+ non-Annotated"

    !!! tip
        Prefer to use the `Annotated` version if possible.

    ```Python hl_lines="13"
    {!> ../../../docs_src/query_params_str_validations/tutorial008.py!}
    ```

## 别名参数

假设你想要查询参数为 `item-query`。

像下面这样：

```
http://127.0.0.1:8000/items/?item-query=foobaritems
```

但是 `item-query` 不是一个有效的 Python 变量名称。

最接近的有效名称是 `item_query`。

但是你仍然要求它在 URL 中必须是 `item-query`...

这时你可以用 `alias` 参数声明一个别名，该别名将用于在 URL 中查找查询参数值：

=== "Python 3.10+"

    ```Python hl_lines="9"
    {!> ../../../docs_src/query_params_str_validations/tutorial009_an_py310.py!}
    ```

=== "Python 3.9+"

    ```Python hl_lines="9"
    {!> ../../../docs_src/query_params_str_validations/tutorial009_an_py39.py!}
    ```

=== "Python 3.6+"

    ```Python hl_lines="10"
    {!> ../../../docs_src/query_params_str_validations/tutorial009_an.py!}
    ```

=== "Python 3.10+ non-Annotated"

    !!! tip
        Prefer to use the `Annotated` version if possible.

    ```Python hl_lines="7"
    {!> ../../../docs_src/query_params_str_validations/tutorial009_py310.py!}
    ```

=== "Python 3.6+ non-Annotated"

    !!! tip
        Prefer to use the `Annotated` version if possible.

    ```Python hl_lines="9"
    {!> ../../../docs_src/query_params_str_validations/tutorial009.py!}
    ```

## 弃用参数

现在假设你不再喜欢此参数。

你不得不将其保留一段时间，因为有些客户端正在使用它，但你希望文档清楚地将其展示为<abbr title ="已过时，建议不要使用它">已弃用</abbr>。

那么将参数 `deprecated=True` 传入 `Query`：

=== "Python 3.10+"

    ```Python hl_lines="19"
    {!> ../../../docs_src/query_params_str_validations/tutorial010_an_py310.py!}
    ```

=== "Python 3.9+"

    ```Python hl_lines="19"
    {!> ../../../docs_src/query_params_str_validations/tutorial010_an_py39.py!}
    ```

=== "Python 3.6+"

    ```Python hl_lines="20"
    {!> ../../../docs_src/query_params_str_validations/tutorial010_an.py!}
    ```

=== "Python 3.10+ non-Annotated"

    !!! tip
        Prefer to use the `Annotated` version if possible.

    ```Python hl_lines="17"
    {!> ../../../docs_src/query_params_str_validations/tutorial010_py310.py!}
    ```

=== "Python 3.6+ non-Annotated"

    !!! tip
        Prefer to use the `Annotated` version if possible.

    ```Python hl_lines="18"
    {!> ../../../docs_src/query_params_str_validations/tutorial010.py!}
    ```

文档将会像下面这样展示它：

<img src="/img/tutorial/query-params-str-validations/image01.png">

## Exclude from OpenAPI

To exclude a query parameter from the generated OpenAPI schema (and thus, from the automatic documentation systems), set the parameter `include_in_schema` of `Query` to `False`:

=== "Python 3.10+"

    ```Python hl_lines="10"
    {!> ../../../docs_src/query_params_str_validations/tutorial014_an_py310.py!}
    ```

=== "Python 3.9+"

    ```Python hl_lines="10"
    {!> ../../../docs_src/query_params_str_validations/tutorial014_an_py39.py!}
    ```

=== "Python 3.6+"

    ```Python hl_lines="11"
    {!> ../../../docs_src/query_params_str_validations/tutorial014_an.py!}
    ```

=== "Python 3.10+ non-Annotated"

    !!! tip
        Prefer to use the `Annotated` version if possible.

    ```Python hl_lines="8"
    {!> ../../../docs_src/query_params_str_validations/tutorial014_py310.py!}
    ```

=== "Python 3.6+ non-Annotated"

    !!! tip
        Prefer to use the `Annotated` version if possible.

    ```Python hl_lines="10"
    {!> ../../../docs_src/query_params_str_validations/tutorial014.py!}
    ```

## 总结

你可以为查询参数声明额外的校验和元数据。

通用的校验和元数据：

* `alias`
* `title`
* `description`
* `deprecated`

特定于字符串的校验：

* `min_length`
* `max_length`
* `regex`

在这些示例中，你了解了如何声明对 `str` 值的校验。

请参阅下一章节，以了解如何声明对其他类型例如数值的校验。
