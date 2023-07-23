# Response Model - Return Type

You can declare the type used for the response by annotating the *path operation function* **return type**.

You can use **type annotations** the same way you would for input data in function **parameters**, you can use Pydantic models, lists, dictionaries, scalar values like integers, booleans, etc.

=== "Python 3.10+"

    ```Python hl_lines="16  21"
    {!> ../../../docs_src/response_model/tutorial001_01_py310.py!}
    ```

=== "Python 3.9+"

    ```Python hl_lines="18  23"
    {!> ../../../docs_src/response_model/tutorial001_01_py39.py!}
    ```

=== "Python 3.6+"

    ```Python hl_lines="18  23"
    {!> ../../../docs_src/response_model/tutorial001_01.py!}
    ```

FastAPI will use this return type to:

* **Validate** the returned data.
    * If the data is invalid (e.g. you are missing a field), it means that *your* app code is broken, not returning what it should, and it will return a server error instead of returning incorrect data. This way you and your clients can be certain that they will receive the data and the data shape expected.
* Add a **JSON Schema** for the response, in the OpenAPI *path operation*.
    * This will be used by the **automatic docs**.
    * It will also be used by automatic client code generation tools.

But most importantly:

* It will **limit and filter** the output data to what is defined in the return type.
    * This is particularly important for **security**, we'll see more of that below.

## `response_model` Parameter

There are some cases where you need or want to return some data that is not exactly what the type declares.

For example, you could want to **return a dictionary** or a database object, but **declare it as a Pydantic model**. This way the Pydantic model would do all the data documentation, validation, etc. for the object that you returned (e.g. a dictionary or database object).

If you added the return type annotation, tools and editors would complain with a (correct) error telling you that your function is returning a type (e.g. a dict) that is different from what you declared (e.g. a Pydantic model).

In those cases, you can use the *path operation decorator* parameter `response_model` instead of the return type.

你可以在任意的*路径操作*中使用 `response_model` 参数：

* `@app.get()`
* `@app.post()`
* `@app.put()`
* `@app.delete()`
* 等等。

=== "Python 3.10+"

    ```Python hl_lines="17  22  24-27"
    {!> ../../../docs_src/response_model/tutorial001_py310.py!}
    ```

=== "Python 3.9+"

    ```Python hl_lines="17  22  24-27"
    {!> ../../../docs_src/response_model/tutorial001_py39.py!}
    ```

=== "Python 3.6+"

    ```Python hl_lines="17  22  24-27"
    {!> ../../../docs_src/response_model/tutorial001.py!}
    ```

!!! note
    注意，`response_model`是「装饰器」方法（`get`，`post` 等）的一个参数。不像之前的所有参数和请求体，它不属于*路径操作函数*。

`response_model` 接收的类型与你将为 Pydantic 模型字段所声明的类型相同，因此它可以是一个 Pydantic 模型，但也可以是一个由 Pydantic 模型组成的 `list`，例如 `List[Item]`。

FastAPI 将使用此 `response_model` 来 do all the data documentation, validation, etc. and also to **convert and filter the output data** to its type declaration.

!!! tip
    If you have strict type checks in your editor, mypy, etc, you can declare the function return type as `Any`.

    That way you tell the editor that you are intentionally returning anything. But FastAPI will still do the data documentation, validation, filtering, etc. with the `response_model`.

### `response_model` Priority

If you declare both a return type and a `response_model`, the `response_model` will take priority and be used by FastAPI.

This way you can add correct type annotations to your functions even when you are returning a type different than the response model, to be used by the editor and tools like mypy. And still you can have FastAPI do the data validation, documentation, etc. using the `response_model`.

You can also use `response_model=None` to disable creating a response model for that *path operation*, you might need to do it if you are adding type annotations for things that are not valid Pydantic fields, you will see an example of that in one of the sections below.

## 返回与输入相同的数据

现在我们声明一个 `UserIn` 模型，它将包含一个明文密码属性。

=== "Python 3.10+"

    ```Python hl_lines="7  9"
    {!> ../../../docs_src/response_model/tutorial002_py310.py!}
    ```

=== "Python 3.6+"

    ```Python hl_lines="9  11"
    {!> ../../../docs_src/response_model/tutorial002.py!}
    ```

!!! info
    To use `EmailStr`, first install <a href="https://github.com/JoshData/python-email-validator" class="external-link" target="_blank">`email_validator`</a>.

    E.g. `pip install email-validator`
    or `pip install pydantic[email]`.

我们正在使用此模型声明输入数据，并使用同一模型声明输出数据：

=== "Python 3.10+"

    ```Python hl_lines="16"
    {!> ../../../docs_src/response_model/tutorial002_py310.py!}
    ```

=== "Python 3.6+"

    ```Python hl_lines="18"
    {!> ../../../docs_src/response_model/tutorial002.py!}
    ```

现在，每当浏览器使用一个密码创建用户时，API 都会在响应中返回相同的密码。

在这个案例中，这可能不算是问题，因为用户自己正在发送密码。

但是，如果我们在其他的*路径操作*中使用相同的模型，则可能会将用户的密码发送给每个客户端。

!!! danger
    永远不要存储用户的明文密码，也不要在响应中发送密码 like this, unless you know all the caveats and you know what you are doing。

## 添加输出模型

相反，我们可以创建一个有明文密码的输入模型和一个没有明文密码的输出模型：

=== "Python 3.10+"

    ```Python hl_lines="9  11  16"
    {!> ../../../docs_src/response_model/tutorial003_py310.py!}
    ```

=== "Python 3.6+"

    ```Python hl_lines="9  11  16"
    {!> ../../../docs_src/response_model/tutorial003.py!}
    ```

这样，即便我们的*路径操作函数*将会返回包含密码的相同输入用户：

=== "Python 3.10+"

    ```Python hl_lines="24"
    {!> ../../../docs_src/response_model/tutorial003_py310.py!}
    ```

=== "Python 3.6+"

    ```Python hl_lines="24"
    {!> ../../../docs_src/response_model/tutorial003.py!}
    ```

...我们已经将 `response_model` 声明为了不包含密码的 `UserOut` 模型：

=== "Python 3.10+"

    ```Python hl_lines="22"
    {!> ../../../docs_src/response_model/tutorial003_py310.py!}
    ```

=== "Python 3.6+"

    ```Python hl_lines="22"
    {!> ../../../docs_src/response_model/tutorial003.py!}
    ```

因此，**FastAPI** 将会负责过滤掉未在输出模型中声明的所有数据（使用 Pydantic）。

### `response_model` or Return Type

In this case, because the two models are different, if we annotated the function return type as `UserOut`, the editor and tools would complain that we are returning an invalid type, as those are different classes.

That's why in this example we have to declare it in the `response_model` parameter.

...but continue reading below to see how to overcome that.

## Return Type and Data Filtering

Let's continue from the previous example. We wanted to **annotate the function with one type** but return something that includes **more data**.

We want FastAPI to keep **filtering** the data using the response model.

In the previous example, because the classes were different, we had to use the `response_model` parameter. But that also means that we don't get the support from the editor and tools checking the function return type.

But in most of the cases where we need to do something like this, we want the model just to **filter/remove** some of the data as in this example.

And in those cases, we can use classes and inheritance to take advantage of function **type annotations** to get better support in the editor and tools, and still get the FastAPI **data filtering**.

=== "Python 3.10+"

    ```Python hl_lines="7-10  13-14  18"
    {!> ../../../docs_src/response_model/tutorial003_01_py310.py!}
    ```

=== "Python 3.6+"

    ```Python hl_lines="9-13  15-16  20"
    {!> ../../../docs_src/response_model/tutorial003_01.py!}
    ```

With this, we get tooling support, from editors and mypy as this code is correct in terms of types, but we also get the data filtering from FastAPI.

How does this work? Let's check that out. 🤓

### Type Annotations and Tooling

First let's see how editors, mypy and other tools would see this.

`BaseUser` has the base fields. Then `UserIn` inherits from `BaseUser` and adds the `password` field, so, it will include all the fields from both models.

We annotate the function return type as `BaseUser`, but we are actually returning a `UserIn` instance.

The editor, mypy, and other tools won't complain about this because, in typing terms, `UserIn` is a subclass of `BaseUser`, which means it's a *valid* type when what is expected is anything that is a `BaseUser`.

### FastAPI Data Filtering

Now, for FastAPI, it will see the return type and make sure that what you return includes **only** the fields that are declared in the type.

FastAPI does several things internally with Pydantic to make sure that those same rules of class inheritance are not used for the returned data filtering, otherwise you could end up returning much more data than what you expected.

This way, you can get the best of both worlds: type annotations with **tooling support** and **data filtering**.

## 在文档中查看

当你查看自动化文档时，你可以检查输入模型和输出模型是否都具有自己的 JSON Schema：

<img src="/img/tutorial/response-model/image01.png">

并且两种模型都将在交互式 API 文档中使用：

<img src="/img/tutorial/response-model/image02.png">

## Other Return Type Annotations

There might be cases where you return something that is not a valid Pydantic field and you annotate it in the function, only to get the support provided by tooling (the editor, mypy, etc).

### Return a Response Directly

The most common case would be [returning a Response directly as explained later in the advanced docs](../advanced/response-directly.md){.internal-link target=_blank}.

```Python hl_lines="8  10-11"
{!> ../../../docs_src/response_model/tutorial003_02.py!}
```

This simple case is handled automatically by FastAPI because the return type annotation is the class (or a subclass) of `Response`.

And tools will also be happy because both `RedirectResponse` and `JSONResponse` are subclasses of `Response`, so the type annotation is correct.

### Annotate a Response Subclass

You can also use a subclass of `Response` in the type annotation:

```Python hl_lines="8-9"
{!> ../../../docs_src/response_model/tutorial003_03.py!}
```

This will also work because `RedirectResponse` is a subclass of `Response`, and FastAPI will automatically handle this simple case.

### Invalid Return Type Annotations

But when you return some other arbitrary object that is not a valid Pydantic type (e.g. a database object) and you annotate it like that in the function, FastAPI will try to create a Pydantic response model from that type annotation, and will fail.

The same would happen if you had something like a <abbr title='A union between multiple types means "any of these types".'>union</abbr> between different types where one or more of them are not valid Pydantic types, for example this would fail 💥:

=== "Python 3.10+"

    ```Python hl_lines="8"
    {!> ../../../docs_src/response_model/tutorial003_04_py310.py!}
    ```

=== "Python 3.6+"

    ```Python hl_lines="10"
    {!> ../../../docs_src/response_model/tutorial003_04.py!}
    ```

...this fails because the type annotation is not a Pydantic type and is not just a single `Response` class or subclass, it's a union (any of the two) between a `Response` and a `dict`.

### Disable Response Model

Continuing from the example above, you might not want to have the default data validation, documentation, filtering, etc. that is performed by FastAPI.

But you might want to still keep the return type annotation in the function to get the support from tools like editors and type checkers (e.g. mypy).

In this case, you can disable the response model generation by setting `response_model=None`:

=== "Python 3.10+"

    ```Python hl_lines="7"
    {!> ../../../docs_src/response_model/tutorial003_05_py310.py!}
    ```

=== "Python 3.6+"

    ```Python hl_lines="9"
    {!> ../../../docs_src/response_model/tutorial003_05.py!}
    ```

This will make FastAPI skip the response model generation and that way you can have any return type annotations you need without it affecting your FastAPI application. 🤓

## 响应模型编码参数

你的响应模型可以具有默认值，例如：

=== "Python 3.10+"

    ```Python hl_lines="9  11-12"
    {!> ../../../docs_src/response_model/tutorial004_py310.py!}
    ```

=== "Python 3.9+"

    ```Python hl_lines="11  13-14"
    {!> ../../../docs_src/response_model/tutorial004_py39.py!}
    ```

=== "Python 3.6+"

    ```Python hl_lines="11  13-14"
    {!> ../../../docs_src/response_model/tutorial004.py!}
    ```

* `description: Union[str, None] = None`（在 Python 3.10 中则是 `str | None = None`）具有默认值 `None`。
* `tax: float = 10.5` 具有默认值 `10.5`.
* `tags: List[str] = []` 具有一个空列表作为默认值： `[]`.

但如果它们并没有存储实际的值，你可能想从结果中忽略它们的默认值。

举个例子，当你在 NoSQL 数据库中保存了具有许多可选属性的模型，但你又不想发送充满默认值的很长的 JSON 响应。

### 使用 `response_model_exclude_unset` 参数

你可以设置*路径操作装饰器*的 `response_model_exclude_unset=True` 参数：

=== "Python 3.10+"

    ```Python hl_lines="22"
    {!> ../../../docs_src/response_model/tutorial004_py310.py!}
    ```

=== "Python 3.9+"

    ```Python hl_lines="24"
    {!> ../../../docs_src/response_model/tutorial004_py39.py!}
    ```

=== "Python 3.6+"

    ```Python hl_lines="24"
    {!> ../../../docs_src/response_model/tutorial004.py!}
    ```

然后响应中将不会包含那些默认值，而是仅有实际设置的值。

因此，如果你向*路径操作*发送 ID 为 `foo` 的商品的请求，则响应（不包括默认值）将为：

```JSON
{
    "name": "Foo",
    "price": 50.2
}
```

!!! info
    FastAPI 通过 Pydantic 模型的 `.dict()` 配合 <a href="https://pydantic-docs.helpmanual.io/usage/exporting_models/#modeldict" class="external-link" target="_blank">该方法的 `exclude_unset` 参数</a> 来实现此功能。

!!! info
    你还可以使用：

    * `response_model_exclude_defaults=True`
    * `response_model_exclude_none=True`

    参考 <a href="https://pydantic-docs.helpmanual.io/usage/exporting_models/#modeldict" class="external-link" target="_blank">Pydantic 文档</a> 中对 `exclude_defaults` 和 `exclude_none` 的描述。

#### 默认值字段有实际值的数据

但是，如果你的数据在具有默认值的模型字段中有实际的值，例如 ID 为 `bar` 的项：

```Python hl_lines="3  5"
{
    "name": "Bar",
    "description": "The bartenders",
    "price": 62,
    "tax": 20.2
}
```

这些值将包含在响应中。

#### 具有与默认值相同值的数据

如果数据具有与默认值相同的值，例如 ID 为 `baz` 的项：

```Python hl_lines="3  5-6"
{
    "name": "Baz",
    "description": None,
    "price": 50.2,
    "tax": 10.5,
    "tags": []
}
```

即使 `description`、`tax` 和 `tags` 具有与默认值相同的值，FastAPI 足够聪明 (实际上是 Pydantic 足够聪明) 去认识到这一点，它们的值被显式地所设定（而不是取自默认值）。

因此，它们将包含在 JSON 响应中。

!!! tip
    请注意默认值可以是任何值，而不仅是`None`。

    它们可以是一个列表（`[]`），一个值为 `10.5`的 `float`，等等。

### `response_model_include` 和 `response_model_exclude`

你还可以使用*路径操作装饰器*的 `response_model_include` 和 `response_model_exclude` 参数。

它们接收一个由属性名称 `str` 组成的 `set` 来包含（忽略其他的）或者排除（包含其他的）这些属性。

如果你只有一个 Pydantic 模型，并且想要从输出中移除一些数据，则可以使用这种快捷方法。

!!! tip
    但是依然建议你使用上面提到的主意，使用多个类而不是这些参数。

    这是因为即使使用 `response_model_include` 或 `response_model_exclude` 来省略某些属性，在应用程序的 OpenAPI 定义（和文档）中生成的 JSON Schema 仍将是完整的模型。

    这也适用于作用类似的 `response_model_by_alias`。

=== "Python 3.10+"

    ```Python hl_lines="29  35"
    {!> ../../../docs_src/response_model/tutorial005_py310.py!}
    ```

=== "Python 3.6+"

    ```Python hl_lines="31  37"
    {!> ../../../docs_src/response_model/tutorial005.py!}
    ```

!!! tip
    `{"name", "description"}` 语法创建一个具有这两个值的 `set`。

    等同于 `set(["name", "description"])`。

#### 使用 `list` 而不是 `set`

如果你忘记使用 `set` 而是使用 `list` 或 `tuple`，FastAPI 仍会将其转换为 `set` 并且正常工作：

=== "Python 3.10+"

    ```Python hl_lines="29  35"
    {!> ../../../docs_src/response_model/tutorial006_py310.py!}
    ```

=== "Python 3.6+"

    ```Python hl_lines="31  37"
    {!> ../../../docs_src/response_model/tutorial006.py!}
    ```

## 总结

使用*路径操作装饰器*的 `response_model` 参数来定义响应模型，特别是确保私有数据被过滤掉。

使用 `response_model_exclude_unset` 来仅返回显式设定的值。
