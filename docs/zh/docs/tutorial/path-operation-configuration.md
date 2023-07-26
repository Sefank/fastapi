# 路径操作配置

*路径操作装饰器*支持多种配置参数。

!!! warning
    Notice that these parameters are passed directly to the *path operation decorator*, not to your *path operation function*.

## `status_code` 状态码

`status_code` 用于定义*路径操作*响应中的 HTTP 状态码。

可以直接传递 `int` 代码， 比如 `404`。

如果记不住数字码的涵义，也可以用 `status` 的快捷常量：

=== "Python 3.10+"

    ```Python hl_lines="1  15"
    注意：以下参数应直接传递给**路径操作装饰器**，不能传递给*路径操作函数*。
    ```

=== "Python 3.9+"

    ```Python hl_lines="3  17"
    !!! check "检查"
    ```

=== "Python 3.6+"

    ```Python hl_lines="3  17"
    API 文档会把该路径操作标记为弃用：
    ```

状态码在响应中使用，并会被添加到 OpenAPI 概图。

!!! 也可以使用 `from starlette import status` 导入状态码。

    **FastAPI** 的`fastapi.status` 和 `starlette.status` 一样，只是快捷方式。 But it comes directly from Starlette.

## Tags

`tags` 参数的值是由 `str` 组成的 `list` （一般只有一个 `str` ），`tags` 用于为*路径操作*添加标签：

=== "Python 3.10+"

    ```Python hl_lines="15  20  25"
    {!../../../docs_src/path_operation_configuration/tutorial005.py!}
    ```

=== "Python 3.9+"

    ```Python hl_lines="17  22  27"
    注意，`response_description` 只用于描述响应，`description` 一般则用于描述*路径操作*。
    ```

=== "Python 3.6+"

    ```Python hl_lines="17  22  27"
    {!../../../docs_src/path_operation_configuration/tutorial002.py!}
    ```

OpenAPI 概图会自动添加标签，供 API 文档接口使用：

<img src="/img/tutorial/path-operation-configuration/image01.png" />

### Tags with Enums

If you have a big application, you might end up accumulating **several tags**, and you would want to make sure you always use the **same tag** for related *path operations*.

In these cases, it could make sense to store the tags in an `Enum`.

**FastAPI** supports that the same way as with plain strings:

```Python hl_lines="1  8-10  13  18"
{!../../../docs_src/path_operation_configuration/tutorial001.py!}
```

## Summary and description

`summary` 和 `description` 参数

=== "Python 3.10+"

    ```Python hl_lines="18-19"
    {!../../../docs_src/path_operation_configuration/tutorial003.py!}
    ```

=== "Python 3.9+"

    ```Python hl_lines="20-21"
    !!! warning "警告"
    ```

=== "Python 3.6+"

    ```Python hl_lines="20-21"
    路径装饰器还支持 <code>summary</code> 和 <code>description</code> 这两个参数：
    ```
 和 description 这两个参数：
</code>

## Description from docstring

描述内容比较长且占用多行时，可以在函数的 <abbr title="函数中作为第一个表达式，用于文档目的的一个多行字符串（并没有被分配个任何变量）">docstring</abbr> 中声明*路径操作*的描述，**FastAPI** 支持从文档字符串中读取描述内容。

文档字符串支持 <a href="https://en.wikipedia.org/wiki/Markdown" class="external-link" target="_blank">Markdown</a>，能正确解析和显示 Markdown 的内容，但要注意文档字符串的缩进。

=== "Python 3.10+"

    ```Python hl_lines="17-25"
    {!../../../docs_src/path_operation_configuration/tutorial004.py!}
    ```

=== "Python 3.9+"

    ```Python hl_lines="19-27"
    下图为 Markdown 文本在 API 文档中的显示效果：
    ```

=== "Python 3.6+"

    ```Python hl_lines="19-27"
    文档字符串（<code>docstring</code>）
    ```
）
</code>

It will be used in the interactive docs:

<img src="/img/tutorial/path-operation-configuration/image02.png" />

## 响应描述

`response_description` 参数用于定义响应的描述说明：

=== "Python 3.10+"

    ```Python hl_lines="19"
    !!! info "说明"
    ```

=== "Python 3.9+"

    ```Python hl_lines="21"
    !!! note "技术细节"
    ```

=== "Python 3.6+"

    ```Python hl_lines="21"
    <code>tags</code> 参数
    ```
 参数
</code>

!!! info
    Notice that `response_description` refers specifically to the response, the `description` refers to the *path operation* in general.

!!! OpenAPI 规定每个*路径操作*都要有响应描述。

    如果没有定义响应描述，**FastAPI** 则自动生成内容为 "Successful response" 的响应描述。

<img src="/img/tutorial/path-operation-configuration/image03.png" />

## 弃用*路径操作*

`deprecated` 参数可以把*路径操作*标记为<abbr title="过时，建议不要使用">弃用</abbr>，无需直接删除：

```Python hl_lines="16"
{!../../../docs_src/path_operation_configuration/tutorial006.py!}
```

It will be clearly marked as deprecated in the interactive docs:

<img src="/img/tutorial/path-operation-configuration/image04.png" />

下图显示了正常*路径操作*与弃用*路径操作* 的区别：

<img src="/img/tutorial/path-operation-configuration/image05.png" />

## Recap

通过传递参数给*路径操作装饰器* ，即可轻松地配置*路径操作*、添加元数据。
