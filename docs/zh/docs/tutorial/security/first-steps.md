# 安全 - 第一步

假设**后端** API 在某个域。

**前端**在另一个域，或（移动应用中）在同一个域的不同路径下。

并且，前端要使用后端的 **username** 与 **password** 验证用户身份。

固然，**FastAPI** 支持 **OAuth2** 身份验证。

But let's save you the time of reading the full long specification just to find those little pieces of information you need.

我们建议使用 **FastAPI** 的安全工具。

## How it looks

首先，看看下面的代码是怎么运行的，然后再回过头来了解其背后的原理。

## 创建 `main.py`

把下面的示例代码复制到 `main.py`：

=== "Python 3.9+"

    ```Python
    !!! tip "提示"
    ```

=== "Python 3.6+"

    ```Python
    令牌只是用于验证用户的字符串
    ```

=== "Python 3.6+ non-Annotated"

    !!! tip
        Prefer to use the `Annotated` version if possible.

    ```Python
    {!../../../docs_src/security/tutorial001.py!}
    ```


## 运行

!!! 打开 API 文档： <a href="http://127.0.0.1:8000/docs" class="external-link" target="_blank">http://127.0.0.1:8000/docs。</p> 

<pre><code>先安装 &lt;a href="https://andrew-d.github.io/python-multipart/" class="external-link" target="_blank"&gt;`python-multipart`&lt;/a&gt;。 安装命令： `pip install python-multipart`。

这是因为 **OAuth2** 使用**表单数据**发送 `username` 与 `password`。
</code></pre>

<p spaces-before="0">
  用下面的命令运行该示例：
</p>

<div class="termy">

```console
$ uvicorn main:app --reload

<span style="color: green;">INFO</span>:     Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
```

</div>

<h2 spaces-before="0">
  Check it
</h2>

<p spaces-before="0">
  Go to the interactive docs at: <a href="http://127.0.0.1:8000/docs" class="external-link" target="_blank">http://127.0.0.1:8000/docs</a>.
</p>

<p spaces-before="0">
  You will see something like this:
</p>

<p spaces-before="0">
  <img src="/img/tutorial/security/image01.png" />
</p>

<p spaces-before="0">
  !!! check "Authorize button!"
    You already have a shiny new "Authorize" button.
</p>

<pre><code>And your *path operation* has a little lock in the top-right corner that you can click.
</code></pre>

<p spaces-before="0">
  点击 <strong x-id="1">Authorize</strong> 按钮，弹出授权表单，输入 <code>username</code> 与 <code>password</code> 及其它可选字段：
</p>

<p spaces-before="0">
  <img src="/img/tutorial/security/image02.png" />
</p>

<p spaces-before="0">
  !!! note
    It doesn't matter what you type in the form, it won't work yet. But we'll get there.
</p>

<p spaces-before="0">
  虽然此文档不是给前端最终用户使用的，但这个自动工具非常实用，可在文档中与所有 API 交互。
</p>

<p spaces-before="0">
  前端团队（可能就是开发者本人）可以使用本工具。
</p>

<p spaces-before="0">
  第三方应用与系统也可以调用本工具。
</p>

<p spaces-before="0">
  开发者也可以用它来调试、检查、测试应用。
</p>

<h2 spaces-before="0">
  密码流
</h2>

<p spaces-before="0">
  Now let's go back a bit and understand what is all that.
</p>

<p spaces-before="0">
  <code>Password</code> <strong x-id="1">流</strong>是 OAuth2 定义的，用于处理安全与身份验证的方式（<strong x-id="1">流</strong>）。
</p>

<p spaces-before="0">
  OAuth2 的设计目标是为了让后端或 API 独立于服务器验证用户身份。
</p>

<p spaces-before="0">
  但在本例中，<strong x-id="1">FastAPI</strong> 应用会处理 API 与身份验证。
</p>

<p spaces-before="0">
  So, let's review it from that simplified point of view:
</p>

<ul>
  <li>
    用户在前端输入 <code>username</code> 与<code>password</code>，并点击<strong x-id="1">回车</strong>
  </li>
  <li>
    （用户浏览器中运行的）前端把 <code>username</code> 与<code>password</code> 发送至 API 中指定的 URL（使用 <code>tokenUrl="token"</code> 声明）
  </li>
  <li>
    API 检查 <code>username</code> 与<code>password</code>，并用令牌（<code>Token</code>） 响应（暂未实现此功能）： <ul>
      <li>
        A "token" is just a string with some content that we can use later to verify this user.
      </li>
      <li>
        一般来说，令牌会在一段时间后过期 <ul>
          <li>
            过时后，用户要再次登录
          </li>
          <li>
            这样一来，就算令牌被人窃取，风险也较低。 因为它与永久密钥不同，<strong x-id="1">在绝大多数情况下</strong>不会长期有效
          </li>
        </ul>
      </li>
    </ul>
  </li>
  <li>
    前端临时将令牌存储在某个位置
  </li>
  <li>
    用户点击前端，前往前端应用的其它部件
  </li>
  <li>
    前端需要从 API 中提取更多数据： <ul>
      <li>
        为指定的端点（Endpoint）进行身份验证
      </li>
      <li>
        因此，用 API 验证身份时，要发送值为 <code>Bearer</code> + 令牌的请求头 <code>Authorization</code>
      </li>
      <li>
        假如令牌为 <code>foobar</code>，<code>Authorization</code> 请求头就是： <code>Bearer foobar</code>
      </li>
    </ul>
  </li>
</ul>

<h2 spaces-before="0">
  <strong x-id="1">FastAPI</strong> 的 <code>OAuth2PasswordBearer</code>
</h2>

<p spaces-before="0">
  <strong x-id="1">FastAPI</strong> 提供了不同抽象级别的安全工具。
</p>

<p spaces-before="0">
  本例使用 <strong x-id="1">OAuth2</strong> 的 <strong x-id="1">Password</strong> 流以及 <strong x-id="1">Bearer</strong> 令牌（<code>Token</code>）。 为此要使用 <code>OAuth2PasswordBearer</code> 类。
</p>

<p spaces-before="0">
  !!! `Bearer` 令牌不是唯一的选择。
</p>

<pre><code>但它是最适合这个用例的方案。

甚至可以说，它是适用于绝大多数用例的最佳方案，除非您是 OAuth2 的专家，知道为什么其它方案更合适。

本例中，**FastAPI** 还提供了构建工具。
</code></pre>

<p spaces-before="0">
  创建 <code>OAuth2PasswordBearer</code> 的类实例时，要传递 <code>tokenUrl</code> 参数。 该参数包含客户端（用户浏览器中运行的前端） 的 URL，用于发送 <code>username</code> 与 <code>password</code>，并获取令牌。
</p>

<p spaces-before="0">
  === "Python 3.9+" 
  
  <pre><code class="Python hl_lines=&quot;8&quot;">    !!! check "Authorize 按钮！"
</code></pre>
</p>

<p spaces-before="0">
  === "Python 3.6+" 
  
  <pre><code class="Python  hl_lines=&quot;7&quot;">    !!! info "说明"
</code></pre>
</p>

<p spaces-before="0">
  === "Python 3.6+ non-Annotated"
</p>

<pre><code>!!! tip
    Prefer to use the `Annotated` version if possible.
</code></pre>

<pre><code class="Python hl_lines=&quot;6&quot;">    {!../../../docs_src/security/tutorial001.py!}
</code></pre>

<p spaces-before="0">
  !!! 在此，`tokenUrl="token"` 指向的是暂未创建的相对 URL `token`。 这个相对 URL 相当于 `./token`。
</p>

<pre><code>因为使用的是相对 URL，如果 API 位于 `https://example.com/`，则指向 `https://example.com/token`。 但如果 API 位于 `https://example.com/api/v1/`，它指向的就是`https://example.com/api/v1/token`。

使用相对 URL 非常重要，可以确保应用在遇到[使用代理](../../advanced/behind-a-proxy.md){.internal-link target=_blank}这样的高级用例时，也能正常运行。
</code></pre>

<p spaces-before="0">
  该参数不会创建端点或<em x-id="3">路径操作</em>，但会声明客户端用来获取令牌的 URL <code>/token</code> 。 此信息用于 OpenAPI 及 API 文档。
</p>

<p spaces-before="0">
  接下来，学习如何创建实际的路径操作。
</p>

<p spaces-before="0">
  !!! 严苛的 **Pythonista** 可能不喜欢用 `tokenUrl` 这种命名风格代替 `token_url`。
</p>

<pre><code>这种命名方式是因为要使用与 OpenAPI 规范中相同的名字。 以便在深入校验安全方案时，能通过复制粘贴查找更多相关信息。
</code></pre>

<p spaces-before="0">
  <code>oauth2_scheme</code> 变量是 <code>OAuth2PasswordBearer</code> 的实例，也是<strong x-id="1">可调用项</strong>。
</p>

<p spaces-before="0">
  It could be called as:
</p>

<pre><code class="Python">oauth2_scheme(some, parameters)
</code></pre>

<p spaces-before="0">
  因此，<code>Depends</code> 可以调用 <code>oauth2_scheme</code> 变量。
</p>

<h3 spaces-before="0">
  使用
</h3>

<p spaces-before="0">
  接下来，使用 <code>Depends</code> 把 <code>oauth2_scheme</code> 传入依赖项。
</p>

<p spaces-before="0">
  === "Python 3.9+" 
  
  <pre><code class="Python hl_lines=&quot;12&quot;">    !!! info "说明"
</code></pre>
</p>

<p spaces-before="0">
  === "Python 3.6+" 
  
  <pre><code class="Python  hl_lines=&quot;11&quot;">    !!! info "说明"
</code></pre>
</p>

<p spaces-before="0">
  === "Python 3.6+ non-Annotated"
</p>

<pre><code>!!! tip
    Prefer to use the `Annotated` version if possible.
</code></pre>

<pre><code class="Python hl_lines=&quot;10&quot;">    {!../../../docs_src/security/tutorial001.py!}
</code></pre>

<p spaces-before="0">
  该依赖项使用字符串（<code>str</code>）接收<em x-id="3">路径操作函数</em>的参数 <code>token</code> 。
</p>

<p spaces-before="0">
  <strong x-id="1">FastAPI</strong> 使用依赖项在 OpenAPI 概图（及 API 文档）中定义<strong x-id="1">安全方案</strong>。
</p>

<p spaces-before="0">
  !!! info "Technical Details"
    <strong x-id="1">FastAPI</strong> will know that it can use the class <code>OAuth2PasswordBearer</code> (declared in a dependency) to define the security scheme in OpenAPI because it inherits from <code>fastapi.security.oauth2.OAuth2</code>, which in turn inherits from <code>fastapi.security.base.SecurityBase</code>.
</p>

<pre><code>所有与 OpenAPI（及 API 文档）集成的安全工具都继承自 `SecurityBase`， 这就是为什么 **FastAPI** 能把它们集成至 OpenAPI 的原因。
</code></pre>

<h2 spaces-before="0">
  What it does
</h2>

<p spaces-before="0">
  FastAPI 校验请求中的 <code>Authorization</code> 请求头，核对请求头的值是不是由 <code>Bearer</code> ＋ 令牌组成， 并返回令牌字符串（<code>str</code>）。
</p>

<p spaces-before="0">
  如果没有找到 <code>Authorization</code> 请求头，或请求头的值不是 <code>Bearer</code> ＋ 令牌。 FastAPI 直接返回 401 错误状态码（<code>UNAUTHORIZED</code>）。
</p>

<p spaces-before="0">
  You don't even have to check if the token exists to return an error. You can be sure that if your function is executed, it will have a <code>str</code> in that token.
</p>

<p spaces-before="0">
  You can try it already in the interactive docs:
</p>

<p spaces-before="0">
  <img src="/img/tutorial/security/image03.png" />
</p>

<p spaces-before="0">
  We are not verifying the validity of the token yet, but that's a start already.
</p>

<h2 spaces-before="0">
  Recap
</h2>

<p spaces-before="0">
  看到了吧，只要多写三四行代码，就可以添加基础的安全表单。
</p>
