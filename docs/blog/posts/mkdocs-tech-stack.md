---
date: 2025-02-23
authors: 
  - Jimmy
categories:
  - Mkdocs
  - Tech
---
# Note 搭建维护小记

本文记录了在搭建Note时使用的技术栈，再此记录便于后续更新维护。

由于最近心血来潮，把整个Note的技术架构做了一次升级，花了巨大的时间做了很多对以前配置的兼容性更改，以更好的支持新feature。
所以这次以后并不打算做进一步的升级了，除非是什么根本性的更新，之后可以目前版本的基础上自己进行开发更新降低兼容难度，而且可以更好的满足custom的需求。
<!-- more -->
## 基础配置

首先先把使用到的基础包总体列一下。

|Package|Version|备注|
|---|---|---|
|mkdocs|1.6.1|Fork from Offical [mkdocs](https://github.com/mkdocs/mkdocs/commit/bb7e8b62185b11d9f59bb7f50b13c15134f62f8a)|
|mkdocs-material|9.6.5|Fork from Offical [Material](https://github.com/squidfunk/mkdocs-material/commit/7445b2aa608fd1312914103de10949b970b49142)|
|mkdocs-material-extensions|1.3.1|Same as the above|
|mkdocs-glightbox|0.4.0|Fork from Offical [mkdocs-glightbox](https://github.com/blueswen/mkdocs-glightbox/commit/7f68f19556c8d91eb45bed2f5e3b93f9d0b4e591)|
|mkdocs-git-revision-date-localized-plugin|1.3.0|Fork from Offical [mkdocs-git-revision-date-localized-plugin](https://github.com/timvink/mkdocs-git-revision-date-localized-plugin/commit/2e7646ee3405d8793e5ebb83eb45f7aa9407b205)|
|mkdocs-encryptcontent-plugin|2.2.1|Fork from [TonyCrane](https://github.com/TonyCrane/mkdocs-toolchain)|
|mkdocs-changelog-plugin|0.1.3|Fork from [TonyCrane](https://github.com/TonyCrane/mkdocs-changelog-plugin)|
|mkdocs-heti-plugin|0.1.6|Fork from [TonyCrane](https://github.com/TonyCrane/mkdocs-heti-plugin)|
|mkdocs-statistics-plugin|0.1.4|Fork from [TonyCrane](https://github.com/TonyCrane/mkdocs-statistics-plugin)|
|mkdocs-tikzautomata-plugin|0.0.1|Fork from [TonyCrane](https://github.com/TonyCrane/mkdocs-toolchain)|
|mkdocs-toc-plugin|0.0.2|Fork from [TonyCrane](https://github.com/TonyCrane/mkdocs-toolchain)|

> **glightbox**: 实现图片点击放大查看
>
> **git-revision-date-localized-plugin**：能够根据git log自动跟踪文档修改和创建时间
>
> **encryptcontent-plugin**：实现文档页面内容加密
>
> **changelog-plugin**：好看的更新记录插件
>
> **heti-plugin**：自动实现中西文之间的空格
>
> **statistics-plugin**：实现所有文档内容字数等统计
>
> **tikzautomata-plugin**：在markdown中直接编写tikz并嵌入svg
>
> **toc-plugin**：实现自定义的table of content
>

## 首页UI配置

参照[pokemon-cards-css](https://github.com/simeydotme/pokemon-cards-css/)项目，简单制作了卡片UI，能够将就着看。

??? quote "设计展示"
    ![light](https://cdn.jsdelivr.net/gh/Jimmy-Bots/note/docs/assets/cards/back.light.png)
    ![dark](https://cdn.jsdelivr.net/gh/Jimmy-Bots/note/docs/assets/cards/back.png)

其他部分设计参考了[鹤翔万里的笔记本](https://note.tonycrane.cc/)

## 文档时间记录的额外配置

对于部分加密文件，其没有被git所记录，所以时间依靠手动更新，需要重写`content.html`文件。

利用`page.meta.data`和`page.meta.revision_data`两个参数来手动指定创建时间和修改时间。

???+ note "重写部分"
    ```html
    {% if page and page.meta and (
      page.meta.git_revision_date_localized or
      page.meta.revision_date
    ) %}
      {% if page and page.meta %} 
        {% if page.meta.hide and "date" in page.meta.hide %}
        {% elif page.meta.data and page.meta.revision_data %}
          <aside class="md-source-file">
            <span class="md-source-file__fact">
            <span class="md-icon" title="最后更新">
            <svg viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path d="M21 13.1c-.1 0-.3.1-.4.2l-1 1 2.1 2.1 1-1c.2-.2.2-.6 0-.8l-1.3-1.3c-.1-.1-.2-.2-.4-.2m-1.9 1.8-6.1 6V23h2.1l6.1-6.1zM12.5 7v5.2l4 2.4-1 1L11 13V7zM11 21.9c-5.1-.5-9-4.8-9-9.9C2 6.5 6.5 2 12 2c5.3 0 9.6 4.1 10 9.3-.3-.1-.6-.2-1-.2s-.7.1-1 .2C19.6 7.2 16.2 4 12 4c-4.4 0-8 3.6-8 8 0 4.1 3.1 7.5 7.1 7.9l-.1.2z"></path></svg>
            </span>
            <span class="git-revision-date-localized-plugin git-revision-date-localized-plugin-custom">{{ page.meta.revision_data }}</span>
            </span>
            <span class="md-source-file__fact">
            <span class="md-icon" title="创建日期">
            <svg viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path d="M14.47 15.08 11 13V7h1.5v5.25l3.08 1.83c-.41.28-.79.62-1.11 1m-1.39 4.84c-.36.05-.71.08-1.08.08-4.42 0-8-3.58-8-8s3.58-8 8-8 8 3.58 8 8c0 .37-.03.72-.08 1.08.69.1 1.33.32 1.92.64.1-.56.16-1.13.16-1.72 0-5.5-4.5-10-10-10S2 6.5 2 12s4.47 10 10 10c.59 0 1.16-.06 1.72-.16-.32-.59-.54-1.23-.64-1.92M18 15v3h-3v2h3v3h2v-3h3v-2h-3v-3z"></path></svg>
            </span>
            <span class="git-revision-date-localized-plugin git-revision-date-localized-plugin-custom">{{ page.meta.data }}</span>
            </span>
          </aside>
        {% else %}
          {% include "partials/source-file.html" %}
        {% endif %}
      {% endif %}
    {% endif %}
    ```

## 修复enrypt-plugin

在使用加密插件的时候，通过使用以下配置来同时加密toc

```yaml
encrypted_something:
  mkdocs-encrypted-toc: [nav, class]
```

然而发现当输入正确密码后无法正确显示toc，于是进一步调试发现toc部分并未被正确加密，而是单纯被设置为`display: none`,在html源码中能找到未加密的toc明文。继续阅读源码，可以发现在解密js中有如下代码。

```javascript
if (html_item[i]) {
    var parts = html_item[i].innerHTML.split(';');
    // decrypt it
    if (parts[0], parts[1], parts[2]) {
        var content = decrypt_content(password_input.value, parts[0], parts[1], parts[2]);
    };
    if (content) {
        // success; display the decrypted content
        html_item[i].innerHTML = content;
        html_item[i].style.display = null;
        // any post processing on the decrypted content should be done here
    };
}
```

其中`content`有无内容决定了是否显示被隐藏的内容，但是由于之前加密中放的是明文内容，导致解密后的`content==undefined`，所以导致了无法显示。

于是追本溯源，发现在原来的encrypt-plugin中有这样一段代码存在问题，并没有正确使用`beatifulSoup`这个库来导致的。

```python
item.contents = [
  content for content in item.contents if content not in ['\n', ' ']
]
# Merge the content in case there are several elements
if len(item.contents) > 1:
  merge_item = ''.join([str(s) for s in item.contents])
elif len(item.contents) == 1:
  merge_item = item.contents[0]
else:
  merge_item = ""
# Encrypt child items on target tags with page password
cipher_bundle = self.__encrypt_text_aes__(merge_item, str(page.password))
encrypted_content = b';'.join(cipher_bundle).decode('ascii')
# Replace initial content with encrypted one
bs4_encrypted_content = BeautifulSoup(encrypted_content, 'html.parser')
item.contents = [bs4_encrypted_content] # 这里未能正确使用
if item.has_attr('style'):
  if isinstance(item['style'], list):
    item['style'].append("display:none")
  else:
    item['style'] = item['class'] + "display:none"
else:
  item['style'] = "display:none"
```

首先`item`的类型是`bs4.element.Tag`，我们可以知道它有一个`content`属性用来存放其中存储的内容，在加密源码中我们可以发现，它直接对`content`进行了赋值操作，这是一个有问题的操作，单纯的赋值并没有对tree进行任何改变，所以对最终的html没有任何影响。于是看一下`tag`的源码

```python
def clear(self, decompose=False):
  """Wipe out all children of this PageElement by calling extract()
      on them.

  :param decompose: If this is True, decompose() (a more
      destructive method) will be called instead of extract().
  """
  if decompose:
    for element in self.contents[:]:
      if isinstance(element, Tag):
        element.decompose()
      else:
        element.extract()
  else:
    for element in self.contents[:]:
      element.extract()

@string.setter
  def string(self, string):
    """Replace this PageElement's contents with `string`."""
    self.clear()
    self.append(string.__class__(string))
```

可以发现对`contents`的处理都是通过其他方式实现的，所以可以猜测为`contents`可能只能算是个只读变量，后面继续深扒了`append`，而它又是通过`insert`，最终可以发现是在所有操作结束后把结果填入`contents`中，因此验证了之前的猜测。

知道了原因，所以修复也很容易，只需要借用一下`tag.string.setter`的方法，即可完成原有需求。

## 小结

一下子就写了这么点，其实也就为了记录一下这次的升级过程中遇到的一些问题，也不知道该写啥了。可能还有其他小问题忘了，想起来了再慢慢补上，后面要是又有什么新问题也能补上。
