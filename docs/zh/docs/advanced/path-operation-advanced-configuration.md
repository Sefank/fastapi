# 路径操作的高级配置

## OpenAPI 的 operationId

!!! !!! warning
    如果你并非 OpenAPI 的「专家」，你可能不需要这部分内容。

你可以在路径操作中通过参数 `operation_id` 设置要使用的 OpenAPI `operationId`。

You would have to make sure that it is unique for each operation.

```Python hl_lines="6"
{!../../../docs_src/path_operation_advanced_configuration/tutorial001.py!}
```

### 使用 *路径操作函数* 的函数名作为 operationId

如果你想用你的 API 的函数名作为 `operationId` 的名字，你可以遍历一遍 API 的函数名，然后使用他们的 `APIRoute.name` 重写每个 *路径操作* 的 `operation_id`。

你应该在添加了所有 *路径操作* 之后执行此操作。

```Python hl_lines="2  12-21  24"
{!../../../docs_src/path_operation_advanced_configuration/tutorial002.py!}
```

!!! !!! tip
    如果你手动调用 `app.openapi()`，你应该在此之前更新 `operationId`。

!!! !!! warning
    如果你这样做，务必确保你的每个 *路径操作函数* 的名字唯一。

    即使它们在不同的模块中（Python 文件）。

## 从 OpenAPI 中排除

使用参数 `include_in_schema` 并将其设置为 `False` ，来从生成的 OpenAPI 方案中排除一个 *路径操作*（这样一来，就从自动化文档系统中排除掉了）。

```Python hl_lines="6"
{!../../../docs_src/path_operation_advanced_configuration/tutorial003.py!}
```

## docstring 的高级描述

你可以限制 *路径操作函数* 的 `docstring` 中用于 OpenAPI 的行数。

添加一个 `\f` （一个「换页」的转义字符）可以使 **FastAPI** 在那一位置截断用于 OpenAPI 的输出。

剩余部分不会出现在文档中，但是其他工具（比如 Sphinx）可以使用剩余部分。

```Python hl_lines="19-29"
{!../../../docs_src/path_operation_advanced_configuration/tutorial004.py!}
```

## Additional Responses

You probably have seen how to declare the `response_model` and `status_code` for a *path operation*.

That defines the metadata about the main response of a *path operation*.

You can also declare additional responses with their models, status codes, etc.

There's a whole chapter here in the documentation about it, you can read it at [Additional Responses in OpenAPI](./additional-responses.md){.internal-link target=_blank}.

## OpenAPI Extra

When you declare a *path operation* in your application, **FastAPI** automatically generates the relevant metadata about that *path operation* to be included in the OpenAPI schema.

!!! note "Technical details"
    In the OpenAPI specification it is called the <a href="https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.0.3.md#operation-object" class="external-link" target="_blank">Operation Object</a>.

It has all the information about the *path operation* and is used to generate the automatic documentation.

It includes the `tags`, `parameters`, `requestBody`, `responses`, etc.

This *path operation*-specific OpenAPI schema is normally generated automatically by **FastAPI**, but you can also extend it.

!!! tip
    This is a low level extension point.

    If you only need to declare additional responses, a more convenient way to do it is with [Additional Responses in OpenAPI](./additional-responses.md){.internal-link target=_blank}.

You can extend the OpenAPI schema for a *path operation* using the parameter `openapi_extra`.

### OpenAPI Extensions

This `openapi_extra` can be helpful, for example, to declare [OpenAPI Extensions](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.0.3.md#specificationExtensions):

```Python hl_lines="6"
{!../../../docs_src/path_operation_advanced_configuration/tutorial005.py!}
```

If you open the automatic API docs, your extension will show up at the bottom of the specific *path operation*.

<img src="/img/tutorial/path-operation-advanced-configuration/image01.png" />

And if you see the resulting OpenAPI (at `/openapi.json` in your API), you will see your extension as part of the specific *path operation* too:

```JSON hl_lines="22"
{
    "openapi": "3.1.0",
    "info": {
        "title": "FastAPI",
        "version": "0.1.0"
    },
    "paths": {
        "/items/": {
            "get": {
                "summary": "Read Items",
                "operationId": "read_items_items__get",
                "responses": {
                    "200": {
                        "description": "Successful Response",
                        "content": {
                            "application/json": {
                                "schema": {}
                            }
                        }
                    }
                },
                "x-aperture-labs-portal": "blue"
            }
        }
    }
}
```

### Custom OpenAPI *path operation* schema

The dictionary in `openapi_extra` will be deeply merged with the automatically generated OpenAPI schema for the *path operation*.

So, you could add additional data to the automatically generated schema.

For example, you could decide to read and validate the request with your own code, without using the automatic features of FastAPI with Pydantic, but you could still want to define the request in the OpenAPI schema.

You could do that with `openapi_extra`:

```Python hl_lines="20-37  39-40"
{!../../../docs_src/path_operation_advanced_configuration/tutorial006.py!}
```

In this example, we didn't declare any Pydantic model. In fact, the request body is not even <abbr title="converted from some plain format, like bytes, into Python objects">parsed</abbr> as JSON, it is read directly as `bytes`, and the function `magic_data_reader()` would be in charge of parsing it in some way.

Nevertheless, we can declare the expected schema for the request body.

### Custom OpenAPI content type

Using this same trick, you could use a Pydantic model to define the JSON Schema that is then included in the custom OpenAPI schema section for the *path operation*.

And you could do this even if the data type in the request is not JSON.

For example, in this application we don't use FastAPI's integrated functionality to extract the JSON Schema from Pydantic models nor the automatic validation for JSON. In fact, we are declaring the request content type as YAML, not JSON:

=== "Pydantic v2"

    ```Python hl_lines="17-22  24"
    {!> ../../../docs_src/path_operation_advanced_configuration/tutorial007.py!}
    ```

=== "Pydantic v1"

    ```Python hl_lines="17-22  24"
    {!> ../../../docs_src/path_operation_advanced_configuration/tutorial007_pv1.py!}
    ```

!!! info
    In Pydantic version 1 the method to get the JSON Schema for a model was called `Item.schema()`, in Pydantic version 2, the method is called `Item.model_schema_json()`.

Nevertheless, although we are not using the default integrated functionality, we are still using a Pydantic model to manually generate the JSON Schema for the data that we want to receive in YAML.

Then we use the request directly, and extract the body as `bytes`. This means that FastAPI won't even try to parse the request payload as JSON.

And then in our code, we parse that YAML content directly, and then we are again using the same Pydantic model to validate the YAML content:

=== "Pydantic v2"

    ```Python hl_lines="26-33"
    {!> ../../../docs_src/path_operation_advanced_configuration/tutorial007.py!}
    ```

=== "Pydantic v1"

    ```Python hl_lines="26-33"
    {!> ../../../docs_src/path_operation_advanced_configuration/tutorial007_pv1.py!}
    ```

!!! info
    In Pydantic version 1 the method to parse and validate an object was `Item.parse_obj()`, in Pydantic version 2, the method is called `Item.model_validate()`.

!!! tip
    Here we re-use the same Pydantic model.

    But the same way, we could have validated it in some other way.
