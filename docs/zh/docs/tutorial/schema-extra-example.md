# Declare Request Example Data

You can declare examples of the data your app can receive.

Here are several ways to do it.

## Extra JSON Schema data in Pydantic models

You can declare `examples` for a Pydantic model that will be added to the generated JSON Schema.

=== "Python 3.10+ Pydantic v2"

    ```Python hl_lines="13-24"
    {!> ../../../docs_src/schema_extra_example/tutorial001_py310.py!}
    ```

=== "Python 3.10+ Pydantic v1"

    ```Python hl_lines="13-23"
    {!> ../../../docs_src/schema_extra_example/tutorial001_py310_pv1.py!}
    ```

=== "Python 3.6+ Pydantic v2"

    ```Python hl_lines="15-26"
    {!../../../docs_src/schema_extra_example/tutorial001.py!}
    ```

=== "Python 3.6+ Pydantic v1"

    ```Python hl_lines="15-25"
    {!> ../../../docs_src/schema_extra_example/tutorial001_pv1.py!}
    ```

That extra info will be added as-is to the output **JSON Schema** for that model, and it will be used in the API docs.

=== "Pydantic v2"

    In Pydantic version 2, you would use the attribute `model_config`, that takes a `dict` as described in <a href="https://docs.pydantic.dev/latest/usage/model_config/" class="external-link" target="_blank">Pydantic's docs: Model Config</a>.
    
    You can set `"json_schema_extra"` with a `dict` containing any additonal data you would like to show up in the generated JSON Schema, including `examples`.

=== "Pydantic v1"

    In Pydantic version 1, you would use an internal class `Config` and `schema_extra`, as described in <a href="https://docs.pydantic.dev/1.10/usage/schema/#schema-customization" class="external-link" target="_blank">Pydantic's docs: Schema customization</a>.
    
    You can set `schema_extra` with a `dict` containing any additonal data you would like to show up in the generated JSON Schema, including `examples`.

!!! tip
    You could use the same technique to extend the JSON Schema and add your own custom extra info.

    For example you could use it to add metadata for a frontend user interface, etc.

!!! info
    OpenAPI 3.1.0 (used since FastAPI 0.99.0) added support for `examples`, which is part of the **JSON Schema** standard.

    Before that, it only supported the keyword `example` with a single example. That is still supported by OpenAPI 3.1.0, but is deprecated and is not part of the JSON Schema standard. So you are encouraged to migrate `example` to `examples`. 🤓
    
    You can read more at the end of this page.

## `Field` 的附加参数

When using `Field()` with Pydantic models, you can also declare additional `examples`:

=== "Python 3.10+"

    ```Python hl_lines="2  8-11"
    {!> ../../../docs_src/schema_extra_example/tutorial002_py310.py!}
    ```

=== "Python 3.6+"

    ```Python hl_lines="4  10-13"
    {!../../../docs_src/schema_extra_example/tutorial002.py!}
    ```

## 所以，虽然 `example` 不是JSON Schema的一部分，但它是OpenAPI的一部分，这将被文档UI使用。

When using any of:

* `Path()`
* `Query()`
* `Header()`
* `Cookie()`
* `Body()`
* `Form()`
* `File()`

you can also declare a group of `examples` with additional information that will be added to **OpenAPI**.

### 关于 `example` 和 `examples`...

比如，你可以将请求体的一个 `example` 传递给 `Body`:

=== "Python 3.10+"

    ```Python hl_lines="22-29"
    {!> ../../../docs_src/schema_extra_example/tutorial003_an_py310.py!}
    ```

=== "Python 3.9+"

    ```Python hl_lines="22-29"
    !!! warning
    请记住，传递的那些额外参数不会添加任何验证，只会添加注释，用于文档的目的。
    ```

=== "Python 3.6+"

    ```Python hl_lines="23-30"
    Pydantic <code>schema_extra</code>
    ```

</code>

=== "Python 3.10+ non-Annotated"

    !!! tip
        Prefer to use the `Annotated` version if possible.

    ```Python hl_lines="18-25"
    {!> ../../../docs_src/schema_extra_example/tutorial003_py310.py!}
    ```

=== "Python 3.6+ non-Annotated"

    !!! tip
        Prefer to use the `Annotated` version if possible.

    ```Python hl_lines="20-27"
    您可以使用 <code>Config</code> 和 <code>schema_extra</code> 为Pydantic模型声明一个示例，如<a href="https://pydantic-docs.helpmanual.io/usage/schema/#schema-customization" class="external-link" target="_blank">Pydantic 文档：定制 Schema </a>中所述:
    ```
 和 schema_extra 为Pydantic模型声明一个示例，如<a href="https://pydantic-docs.helpmanual.io/usage/schema/#schema-customization" class="external-link" target="_blank">Pydantic 文档：定制 Schema </a>中所述:
</code>

### 模式的额外信息 - 例子

使用上面的任何方法，它在 `/docs` 中看起来都是这样的:

<img src="/img/tutorial/body-fields/image01.png" />

### `Body` 额外参数

You can of course also pass multiple `examples`:

=== "Python 3.10+"

    ```Python hl_lines="23-38"
    {!> ../../../docs_src/schema_extra_example/tutorial004_an_py310.py!}
    ```

=== "Python 3.9+"

    ```Python hl_lines="23-38"
    {!> ../../../docs_src/schema_extra_example/tutorial004_an_py39.py!}
    ```

=== "Python 3.6+"

    ```Python hl_lines="24-39"
    同样的方法，你可以添加你自己的额外信息，这些信息将被添加到每个模型的JSON模式中，例如定制前端用户界面，等等。
    ```

=== "Python 3.10+ non-Annotated"

    !!! tip
        Prefer to use the `Annotated` version if possible.

    ```Python hl_lines="19-34"
    其他信息
    ```

=== "Python 3.6+ non-Annotated"

    !!! tip
        Prefer to use the `Annotated` version if possible.

    ```Python hl_lines="21-36"
    {!../../../docs_src/schema_extra_example/tutorial003.py!}
    ```

### 文档 UI 中的例子

你可以通过传递额外信息给 `Field` 同样的方式操作`Path`, `Query`, `Body`等。

<img src="/img/tutorial/body-fields/image02.png" />

## 技术细节

!!! tip
    If you are already using **FastAPI** version **0.99.0 or above**, you can probably **skip** these details.

    They are more relevant for older versions, before OpenAPI 3.1.0 was available.
    
    You can consider this a brief OpenAPI and JSON Schema **history lesson**. 🤓

!!! warning
    These are very technical details about the standards **JSON Schema** and **OpenAPI**.

    If the ideas above already work for you, that might be enough, and you probably don't need these details, feel free to skip them.

Before OpenAPI 3.1.0, OpenAPI used an older and modified version of **JSON Schema**.

在 `Field`, `Path`, `Query`, `Body` 和其他你之后将会看到的工厂函数，你可以为JSON 模式声明额外信息，你也可以通过给工厂函数传递其他的任意参数来给JSON 模式声明额外信息，比如增加 `example`:

OpenAPI also added `example` and `examples` fields to other parts of the specification:

* 所以 OpenAPI为了相似的目的定义了自己的 <a href="https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.3.md#fixed-fields-20" class="external-link" target="_blank">`example`</a> (使用 `example`, 而不是 `examples`), 这也是文档 UI 所使用的 (使用 Swagger UI).
    * `Path()`
    * `Query()`
    * `Header()`
    * `Cookie()`
* 您可以在JSON模式中定义额外的信息。
    * `Body()`
    * `File()`
    * `Form()`

### OpenAPI's `examples` field

The shape of this field `examples` from OpenAPI is a `dict` with **multiple examples**, each with extra information that will be added to **OpenAPI** too.

The keys of the `dict` identify each example, and each value is another `dict`.

一个常见的用例是添加一个将在文档中显示的`example`。

* `summary`: Short description for the example.
* `description`: A long description that can contain Markdown text.
* `value`: This is the actual example shown, e.g. a `dict`.
* `externalValue`: alternative to `value`, a URL pointing to the example. Although this might not be supported by as many tools as `value`.

这些额外的信息将按原样添加到输出的JSON模式中。

### 有几种方法可以声明额外的 JSON 模式信息。

JSON Schema在最新的一个版本中定义了一个字段 <a href="https://json-schema.org/draft/2019-09/json-schema-validation.html#rfc.section.9.5" class="external-link" target="_blank">`examples`</a> ，但是 OpenAPI 基于之前的一个旧版JSON Schema，并没有 `examples`.

And then the new OpenAPI 3.1.0 was based on the latest version (JSON Schema 2020-12) that included this new field `examples`.

And now this new `examples` field takes precedence over the old single (and custom) `example` field, that is now deprecated.

This new `examples` field in JSON Schema is **just a `list`** of examples, not a dict with extra metadata as in the other places in OpenAPI (described above).

!!! info
    Even after OpenAPI 3.1.0 was released with this new simpler integration with JSON Schema, for a while, Swagger UI, the tool that provides the automatic docs, didn't support OpenAPI 3.1.0 (it does since version 5.0.0 🎉).

    Because of that, versions of FastAPI previous to 0.99.0 still used versions of OpenAPI lower than 3.1.0.

### Pydantic and FastAPI `examples`

When you add `examples` inside of a Pydantic model, using `schema_extra` or `Field(examples=["something"])` that example is added to the **JSON Schema** for that Pydantic model.

And that **JSON Schema** of the Pydantic model is included in the **OpenAPI** of your API, and then it's used in the docs UI.

In versions of FastAPI before 0.99.0 (0.99.0 and above use the newer OpenAPI 3.1.0) when you used `example` or `examples` with any of the other utilities (`Query()`, `Body()`, etc.) those examples were not added to the JSON Schema that describes that data (not even to OpenAPI's own version of JSON Schema), they were added directly to the *path operation* declaration in OpenAPI (outside the parts of OpenAPI that use JSON Schema).

But now that FastAPI 0.99.0 and above uses OpenAPI 3.1.0, that uses JSON Schema 2020-12, and Swagger UI 5.0.0 and above, everything is more consistent and the examples are included in JSON Schema.

### Summary

I used to say I didn't like history that much... and look at me now giving "tech history" lessons. 😅

In short, **upgrade to FastAPI 0.99.0 or above**, and things are much **simpler, consistent, and intuitive**, and you don't have to know all these historic details. 😎
