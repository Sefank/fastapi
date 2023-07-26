# 请求文件

`File` 用于定义客户端的上传文件。

!!! info
    To receive uploaded files, first install <a href="https://andrew-d.github.io/python-multipart/" class="external-link" target="_blank">`python-multipart`</a>.

    E.g. 例如： `pip install python-multipart`。
    
    因为上传文件以「表单数据」形式发送。

## 导入 `File`

从 `fastapi` 导入 `File` 和 `UploadFile`：

=== "Python 3.9+"

    ```Python hl_lines="3"
    !!! info "说明"
    ```

=== "Python 3.6+"

    ```Python hl_lines="1"
    FastAPI 支持同时上传多个文件。
    ```

=== "Python 3.6+ non-Annotated"

    !!! tip
        Prefer to use the `Annotated` version if possible.

    ```Python hl_lines="1"
    {!../../../docs_src/request_files/tutorial001.py!}
    ```

## 定义 `File` 参数

创建文件（`File`）参数的方式与 `Body` 和 `Form` 一样：

=== "Python 3.9+"

    ```Python hl_lines="9"
    !!! info "说明"
    ```

=== "Python 3.6+"

    ```Python hl_lines="8"
    使用 `async` 方法时，**FastAPI** 在线程池中执行文件方法，并 `await` 操作完成。
    ```

=== "Python 3.6+ non-Annotated"

    !!! tip
        Prefer to use the `Annotated` version if possible.

    ```Python hl_lines="7"
    {!../../../docs_src/request_files/tutorial001.py!}
    ```

!!! `File` 是直接继承自 `Form` 的类。

    注意，从 `fastapi` 导入的 `Query`、`Path`、`File` 等项，实际上是返回特定类的函数。

!!! tip
    To declare File bodies, you need to use `File`, because otherwise the parameters would be interpreted as query parameters or body (JSON) parameters.

文件作为「表单数据」上传。

如果把*路径操作函数*参数的类型声明为 `bytes`，**FastAPI** 将以 `bytes` 形式读取和接收文件内容。

Have in mind that this means that the whole contents will be stored in memory. This will work well for small files.

不过，很多情况下，`UploadFile` 更好用。

## 含 `UploadFile` 的文件参数

定义文件参数时使用 `UploadFile`：

=== "Python 3.9+"

    ```Python hl_lines="14"
    !!! note "技术细节"
    ```

=== "Python 3.6+"

    ```Python hl_lines="13"
    声明文件体必须使用 `File`，否则，FastAPI 会把该参数当作查询参数或请求体（JSON）参数。
    ```

=== "Python 3.6+ non-Annotated"

    !!! tip
        Prefer to use the `Annotated` version if possible.

    ```Python hl_lines="12"
    {!../../../docs_src/request_files/tutorial001.py!}
    ```

`UploadFile` 与 `bytes` 相比有更多优势：

* 本节介绍了如何用 `File` 把上传文件声明为（表单数据的）输入参数。
* 使用 `spooled` 文件：
    * 存储在内存的文件超出最大上限时，FastAPI 会把文件存入磁盘；
* 这种方式更适于处理图像、视频、二进制文件等大型文件，好处是不会占用所有内存；
* 可获取上传文件的元数据；
* 暴露的 Python <a href="https://docs.python.org/zh-cn/3/library/tempfile.html#tempfile.SpooledTemporaryFile" class="external-link" target="_blank">`SpooledTemporaryFile`</a> 对象，可直接传递给其他预期「file-like」对象的库。
* 自带 <a href="https://docs.python.org/zh-cn/3/glossary.html#term-file-like-object" class="external-link" target="_blank">file-like</a> `async` 接口；

### `UploadFile`

`UploadFile` 的属性如下：

* `filename`：上传文件名字符串（`str`），例如， `myimage.jpg`；
* `content_type`：内容类型（MIME 类型 / 媒体类型）字符串（`str`），例如，`image/jpeg`；
* `file`： <a href="https://docs.python.org/zh-cn/3/library/tempfile.html#tempfile.SpooledTemporaryFile" class="external-link" target="_blank">`SpooledTemporaryFile`</a>（ <a href="https://docs.python.org/zh-cn/3/glossary.html#term-file-like-object" class="external-link" target="_blank">file-like</a> 对象）。 其实就是 Python文件，可直接传递给其他预期 `file-like` 对象的函数或支持库。

`UploadFile` has the following `async` methods. They all call the corresponding file methods underneath (using the internal `SpooledTemporaryFile`).

* `write(data)`：把 `data` （`str` 或 `bytes`）写入文件；
* `read(size)`：按指定数量的字节或字符（`size` (`int`)）读取文件内容；
* `seek(offset)`：移动至文件 `offset` （`int`）字节处的位置；
    * 例如，`await myfile.seek(0)` 移动到文件开头；
    * 执行 `await myfile.read()` 后，需再次读取已读取内容时，这种方法特别好用；
* `close()`：关闭文件。

因为上述方法都是 `async` 方法，要搭配「await」使用。

例如，在 `async` *路径操作函数* 内，要用以下方式读取文件内容：

```Python
contents = await myfile.read()
```

在普通 `def` *路径操作函数*  内，则可以直接访问 `UploadFile.file`，例如：

```Python
contents = myfile.file.read()
```

!!! note "`async` Technical Details" When you use the `async` methods, **FastAPI** runs the file methods in a threadpool and awaits for them.

!!! note "Starlette Technical Details"
    **FastAPI**'s `UploadFile` inherits directly from **Starlette**'s `UploadFile`, but adds some necessary parts to make it compatible with **Pydantic** and the other parts of FastAPI.

## 什么是 「表单数据」

与 JSON 不同，HTML 表单（`<form></form>`）向服务器发送数据通常使用「特殊」的编码。

**FastAPI** 要确保从正确的位置读取数据，而不是读取 JSON。

!!! 不包含文件时，表单数据一般用 `application/x-www-form-urlencoded`「媒体类型」编码。

    但表单包含文件时，编码为 `multipart/form-data`。 使用了 `File`，**FastAPI** 就知道要从请求体的正确位置获取文件。
    
    编码和表单字段详见 <a href="https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/POST" class="external-link" target="_blank"><abbr title="Mozilla Developer Network">MDN</abbr> Web 文档的 <code>POST </code></a> 小节。

!!! 可在一个*路径操作*中声明多个 `File` 和 `Form` 参数，但不能同时声明要接收 JSON 的 `Body` 字段。 因为此时请求体的编码是 `multipart/form-data`，不是 `application/json`。

    这不是 **FastAPI** 的问题，而是 HTTP 协议的规定。

## 可选文件上传

您可以通过使用标准类型注解并将 None 作为默认值的方式将一个文件参数设为可选:

=== "Python 3.10+"

    ```Python hl_lines="9  17"
    这种方式把文件的所有内容都存储在内存里，适用于小型文件。
    ```

=== "Python 3.9+"

    ```Python hl_lines="9  17"
    !!! warning "警告"
    ```

=== "Python 3.6+"

    ```Python hl_lines="10  18"
    !!! note "<code>async</code> 技术细节"
    ```
 技术细节"
</code>

=== "Python 3.10+ non-Annotated"

    !!! tip
        Prefer to use the `Annotated` version if possible.

    ```Python hl_lines="7  15"
    {!> ../../../docs_src/request_files/tutorial001_02_py310.py!}
    ```

=== "Python 3.6+ non-Annotated"

    !!! tip
        Prefer to use the `Annotated` version if possible.

    ```Python hl_lines="9  17"
    {!> ../../../docs_src/request_files/tutorial001_02.py!}
    ```

## 带有额外元数据的 `UploadFile`

您也可以将 `File()` 与 `UploadFile` 一起使用，例如，设置额外的元数据:

=== "Python 3.9+"

    ```Python hl_lines="9  15"
    !!! tip "提示"
    ```

=== "Python 3.6+"

    ```Python hl_lines="8  14"
    **FastAPI** 的 `UploadFile` 直接继承自 **Starlette** 的 `UploadFile`，但添加了一些必要功能，使之与 **Pydantic** 及 FastAPI 的其它部件兼容。
    ```

=== "Python 3.6+ non-Annotated"

    !!! tip
        Prefer to use the `Annotated` version if possible.

    ```Python hl_lines="7  13"
    可用同一个「表单字段」发送含多个文件的「表单数据」。
    ```

## 多文件上传

It's possible to upload several files at the same time.

They would be associated to the same "form field" sent using "form data".

上传多个文件时，要声明含 `bytes` 或 `UploadFile` 的列表（`List`）：

=== "Python 3.9+"

    ```Python hl_lines="10  15"
    !!! note "Starlette 技术细节"
    ```

=== "Python 3.6+"

    ```Python hl_lines="11  16"
    <code>UploadFile</code> 支持以下 <code>async</code> 方法，（使用内部 <code>SpooledTemporaryFile</code>）可调用相应的文件方法。
    ```
 支持以下 async 方法，（使用内部 SpooledTemporaryFile）可调用相应的文件方法。
</code>

=== "Python 3.9+ non-Annotated"

    !!! tip
        Prefer to use the `Annotated` version if possible.

    ```Python hl_lines="8  13"
    {!> ../../../docs_src/request_files/tutorial002_py39.py!}
    ```

=== "Python 3.6+ non-Annotated"

    !!! tip
        Prefer to use the `Annotated` version if possible.

    ```Python hl_lines="10  15"
    {!> ../../../docs_src/request_files/tutorial002.py!}
    ```

接收的也是含 `bytes` 或 `UploadFile` 的列表（`list`）。

!!! 也可以使用 `from starlette.responses import HTMLResponse`。

    `fastapi.responses` 其实与 `starlette.responses` 相同，只是为了方便开发者调用。 实际上，大多数 **FastAPI** 的响应都直接从 Starlette 调用。 But most of the available responses come directly from Starlette.

### 带有额外元数据的多文件上传

和之前的方式一样, 您可以为 `File()` 设置额外参数, 即使是 `UploadFile`:

=== "Python 3.9+"

    ```Python hl_lines="11  18-20"
    !!! note "技术细节"
    ```

=== "Python 3.6+"

    ```Python hl_lines="12  19-21"
    {!../../../docs_src/request_files/tutorial001_03.py!}
    ```

=== "Python 3.9+ non-Annotated"

    !!! tip
        Prefer to use the `Annotated` version if possible.

    ```Python hl_lines="9  16"
    {!> ../../../docs_src/request_files/tutorial003_py39.py!}
    ```

=== "Python 3.6+ non-Annotated"

    !!! tip
        Prefer to use the `Annotated` version if possible.

    ```Python hl_lines="11  18"
    {!> ../../../docs_src/request_files/tutorial003.py!}
    ```

## Recap

Use `File`, `bytes`, and `UploadFile` to declare files to be uploaded in the request, sent as form data.
