# 自定义响应 - HTML，流，文件和其他

**FastAPI** 默认会使用 `JSONResponse` 返回响应。

你可以通过直接返回 `Response` 来重载它，参见 [直接返回响应](response-directly.md){.internal-link target=_blank}。

但如果你直接返回 `Response`，返回数据不会自动转换，也不会自动生成文档（例如，在 HTTP 头 `Content-Type` 中包含特定的「媒体类型」作为生成的 OpenAPI 的一部分）。

你还可以在 *路径操作装饰器* 中声明你想用的 `Response`。

你从 *路径操作函数* 中返回的内容将被放在该 `Response` 中。

并且如果该 `Response` 有一个 JSON 媒体类型（`application/json`），比如使用 `JSONResponse` 或者 `UJSONResponse` 的时候，返回的数据将使用你在路径操作装饰器中声明的任何 Pydantic 的 `response_model` 自动转换（和过滤）。

!!! note
    If you use a response class with no media type, FastAPI will expect your response to have no content, so it will not document the response format in its generated OpenAPI docs.

## 使用 `ORJSONResponse`

例如，如果你需要压榨性能，你可以安装并使用 <a href="https://github.com/ijl/orjson" class="external-link" target="_blank">`orjson`</a> 并将响应设置为 `ORJSONResponse`。

导入你想要使用的 `Response` 类（子类）然后在 *路径操作装饰器* 中声明它。

For large responses, returning a `Response` directly is much faster than returning a dictionary.

This is because by default, FastAPI will inspect every item inside and make sure it is serializable with JSON, using the same [JSON Compatible Encoder](../tutorial/encoder.md){.internal-link target=_blank} explained in the tutorial. This is what allows you to return **arbitrary objects**, for example database models.

使用 `HTMLResponse` 来从 **FastAPI** 中直接返回一个 HTML 响应。

```Python hl_lines="2  7"
{!../../../docs_src/custom_response/tutorial001b.py!}
```

!!! info
    The parameter `response_class` will also be used to define the "media type" of the response.

    在这个例子中，HTTP 头的 `Content-Type` 会被设置成 `application/json`。
    
    并且在 OpenAPI 文档中也会这样记录。

!!! tip
    The `ORJSONResponse` is currently only available in FastAPI, not in Starlette.

## HTML 响应

!!! warning "警告"
    *路径操作函数* 直接返回的 `Response` 不会被 OpenAPI 的文档记录（比如，`Content-Type` 不会被文档记录），并且在自动化交互文档中也是不可见的。

* 导入 `HTMLResponse`。
* 将 `HTMLResponse` 作为你的 *路径操作* 的 `response_class` 参数传入。

```Python hl_lines="2  7"
{!../../../docs_src/custom_response/tutorial002.py!}
```

!!! info
    The parameter `response_class` will also be used to define the "media type" of the response.

    在这个例子中，HTTP 头的 `Content-Type` 会被设置成 `text/html`。
    
    并且在 OpenAPI 文档中也会这样记录。

### 返回一个 `Response`

正如你在 [直接返回响应](response-directly.md){.internal-link target=_blank} 中了解到的，你也可以通过直接返回响应在 *路径操作* 中直接重载响应。

和上面一样的例子，返回一个 `HTMLResponse` 看起来可能是这样：

```Python hl_lines="2  7  19"
{!../../../docs_src/custom_response/tutorial003.py!}
```

!!! warning
    A `Response` returned directly by your *path operation function* won't be documented in OpenAPI (for example, the `Content-Type` won't be documented) and won't be visible in the automatic interactive docs.

!!! info
    Of course, the actual `Content-Type` header, status code, etc, will come from the `Response` object your returned.

### OpenAPI 中的文档和重载 `Response`

如果你想要在函数内重载响应，但是同时在 OpenAPI 中文档化「媒体类型」，你可以使用 `response_class` 参数并返回一个 `Response` 对象。

接着 `response_class` 参数只会被用来文档化 OpenAPI 的 *路径操作*，你的 `Response` 用来返回响应。

#### 直接返回 `HTMLResponse`

比如像这样：

```Python hl_lines="7  21  23"
{!../../../docs_src/custom_response/tutorial004.py!}
```

在这个例子中，函数 `generate_html_response()` 已经生成并返回 `Response` 对象而不是在 `str` 中返回 HTML。

通过返回函数 `generate_html_response()` 的调用结果，你已经返回一个重载 **FastAPI** 默认行为的 `Response` 对象，

但如果你在 `response_class` 中也传入了 `HTMLResponse`，**FastAPI** 会知道如何在 OpenAPI 和交互式文档中使用 `text/html` 将其文档化为 HTML。

<img src="/img/tutorial/custom-response/image01.png" />

## 可用响应

这里有一些可用的响应。

要记得你可以使用 `Response` 来返回任何其他东西，甚至创建一个自定义的子类。

!!! note "Technical Details"
    You could also use `from starlette.responses import HTMLResponse`.

    **FastAPI** 提供了同 `fastapi.responses` 相同的 `starlette.responses` 只是为了方便开发者。 但大多数可用的响应都直接来自 Starlette。

### `Response`

其他全部的响应都继承自主类 `Response`。

你可以直接返回它。

It accepts the following parameters:

* `content` - 一个 `str` 或者 `bytes`。
* `status_code` - 一个 `int` 类型的 HTTP 状态码。
* `headers` - 一个由字符串组成的 `dict`。
* `media_type` - A `str` giving the media type. E.g. `"text/html"`.

FastAPI（实际上是 Starlette）将自动包含 Content-Length 的头。 它还将包含一个基于 media_type 的 Content-Type 头，并为文本类型附加一个字符集。

```Python hl_lines="1  18"
{!../../../docs_src/response_directly/tutorial002.py!}
```

### `HTMLResponse`

如上文所述，接受文本或字节并返回 HTML 响应。

### `PlainTextResponse`

接受文本或字节并返回纯文本响应。

```Python hl_lines="2  7  9"
{!../../../docs_src/custom_response/tutorial005.py!}
```

### `JSONResponse`

接受数据并返回一个 `application/json` 编码的响应。

如上文所述，这是 **FastAPI** 中使用的默认响应。

### `ORJSONResponse`

如上文所述，`ORJSONResponse` 是一个使用 <a href="https://github.com/ijl/orjson" class="external-link" target="_blank">`orjson`</a> 的快速的可选 JSON 响应。

### `UJSONResponse`

`UJSONResponse` 是一个使用 <a href="https://github.com/ultrajson/ultrajson" class="external-link" target="_blank">`ujson`</a> 的可选 JSON 响应。

!!! !!! warning "警告"
    在处理某些边缘情况时，`ujson` 不如 Python 的内置实现那么谨慎。

```Python hl_lines="2  7"
{!../../../docs_src/custom_response/tutorial001.py!}
```

!!! tip
    It's possible that `ORJSONResponse` might be a faster alternative.

### `RedirectResponse`

返回 HTTP 重定向。 默认情况下使用 307 状态代码（临时重定向）。

!!! note "技术细节"
    你也可以使用 `from starlette.responses import HTMLResponse`。

```Python hl_lines="2  9"
{!../../../docs_src/custom_response/tutorial006.py!}
```

---

!!! info "提示"
    参数 `response_class` 也会用来定义响应的「媒体类型」。


```Python hl_lines="2  7  9"
{!../../../docs_src/custom_response/tutorial006b.py!}
```

If you do that, then you can return the URL directly from your *path operation* function.

In this case, the `status_code` used will be the default one for the `RedirectResponse`, which is `307`.

---

`media_type` - 一个给出媒体类型的 `str`，比如 `"text/html"`。

```Python hl_lines="2  7  9"
{!../../../docs_src/custom_response/tutorial006c.py!}
```

### `StreamingResponse`

采用异步生成器或普通生成器/迭代器，然后流式传输响应主体。

```Python hl_lines="2  14"
{!../../../docs_src/custom_response/tutorial007.py!}
```

#### 对类似文件的对象使用 `StreamingResponse`

如果您有类似文件的对象（例如，由 `open()` 返回的对象），则可以在 `StreamingResponse` 中将其返回。

That way, you don't have to read it all first in memory, and you can pass that generator function to the `StreamingResponse`, and return it.

包括许多与云存储，视频处理等交互的库。

```{ .python .annotate hl_lines="2  10-12  14" }
{!../../../docs_src/custom_response/tutorial008.py!}
```

1. This is the generator function. It's a "generator function" because it contains `yield` statements inside.
2. By using a `with` block, we make sure that the file-like object is closed after the generator function is done. So, after it finishes sending the response.
3. This `yield from` tells the function to iterate over that thing named `file_like`. And then, for each part iterated, yield that part as coming from this generator function.

    So, it is a generator function that transfers the "generating" work to something else internally.

    By doing it this way, we can put it in a `with` block, and that way, ensure that it is closed after finishing.

!!! !!! tip "小贴士"
    注意在这里，因为我们使用的是不支持 `async` 和 `await` 的标准 `open()`，我们使用普通的 `def` 声明了路径操作。

### `FileResponse`

异步传输文件作为响应。

与其他响应类型相比，接受不同的参数集进行实例化：

* `path` - 要流式传输的文件的文件路径。
* `headers` - 任何自定义响应头，传入字典类型。
* `media_type` - 给出媒体类型的字符串。 如果未设置，则文件名或路径将用于推断媒体类型。
* `filename` - 如果给出，它将包含在响应的 `Content-Disposition` 中。

文件响应将包含适当的 `Content-Length`，`Last-Modified` 和 `ETag` 的响应头。

```Python hl_lines="2  10"
{!../../../docs_src/custom_response/tutorial009.py!}
```

`Response` 类接受如下参数：

```Python hl_lines="2  8  10"
{!../../../docs_src/custom_response/tutorial009b.py!}
```

In this case, you can return the file path directly from your *path operation* function.

## Custom response class

!!! info "提示"
    当然，实际的 `Content-Type` 头，状态码等等，将来自于你返回的 `Response` 对象。

!!! tip "小贴士"
    `ORJSONResponse` 可能是一个更快的选择。

Let's say you want it to return indented and formatted JSON, so you want to use the orjson option `orjson.OPT_INDENT_2`.

You could create a `CustomORJSONResponse`. The main thing you have to do is create a `Response.render(content)` method that returns the content as `bytes`:

```Python hl_lines="9-14  17"
{!../../../docs_src/custom_response/tutorial009c.py!}
```

Now instead of returning:

```json
{"message": "Hello World"}
```

...this response will return:

```json
{
  "message": "Hello World"
}
```

Of course, you will probably find much better ways to take advantage of this than formatting JSON. 😉

## Default response class

!!! note "说明"
    如果你使用不带有任何媒体类型的响应类，FastAPI 认为你的响应没有任何内容，所以不会在生成的OpenAPI文档中记录响应格式。

!!! info "提示"
    参数 `response_class` 也会用来定义响应的「媒体类型」。

!!! tip "小贴士"
    `ORJSONResponse` 目前只在 FastAPI 中可用，而在 Starlette 中不可用。

```Python hl_lines="2  4"
{!../../../docs_src/custom_response/tutorial010.py!}
```

!!! tip
    You can still override `response_class` in *path operations* as before.

## 额外文档

您还可以使用 `response` 在 OpenAPI 中声明媒体类型和许多其他详细信息：[OpenAPI 中的额外文档](additional-responses.md){.internal-link target=_blank}。
