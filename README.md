# Solid.js for Valve's Panorama UI

尚在实验阶段

魔改了编译，针对 PUI 的 API 进行了优化，比如`createElement`加入属性和父元素两个参数，就极大减少了过多的 API 调用，也解决了无法调用`$.CreatePanelWithProperties`的问题。

## style

对 style 进行了兼容，如果 style 是字符串，在 PUI 里 style 末尾不写分号会弹出错误，所以在编译时会解析自动加上分号。

当 style 是 Object 时，某些属性可以赋值数字，会自动转换成 px，支持列表可查看：[packages/runtime/src/config.ts](https://github.com/RobinCodeX/solid-panorama/blob/master/packages/runtime/src/config.ts#L1)

## class

`class`和`className`两个属性都是支持的，由于 solid.js 提供`classList`属性可按`true | false`动态添加，所以也提供类似的功能，三个属性可同时存在。

```jsx
<Button
    class={current() === 'foo' ? 'selected' : ''}
    onClick={() => setCurrent('foo')}
>
    foo
</Button>

<Button
  class="my-button"
  classList={{selected: current() === 'foo'}}
  onClick={() => setCurrent('foo')}
>foo</Button>
```

## 事件

PUI 的元素事件与 WEB 的完全不同，PUI 是较为简单的，而且绝大多数情况下也不需要向上冒泡，所以不会支持事件冒泡的功能。

对事件进行了优化，事件的回调函数第一个参数是元素本身。

<!-- ## 不支持 Fragment

形如`<> </>`就是 Fragment，由于会在编译时无法识别父元素所以不支持，也许以后可以，至少目前对 babel 的理解太浅，尚不知如何做。 -->

## 支持文本节点

在 HTML 中`<div> Hi </div>`这类情况下`Hi`会渲染成文本节点，也就是 textNode，

文本节点会自动创建 Label，并且默认启用 html 渲染，如果包含 HTML 标签，需要用字符串。

需要注意的是如果文本是以`#`开头的文本，比如`#addon_game_name`，此类会自动调用`$.Localize`，但是不能参杂其它文本。

例如，以下是正确的写法：

```jsx
// 纯文本
<Panel>
    Welcome My Game
</Panel>

// 带HTML标签
<Panel>
    {`<strong>Welcome</strong> My Game`}
</Panel>

// 拼接本地化字段
<Panel>
    <Label text="Welcome" />
    #addon_game_name
    <Label text="(～￣▽￣)～" />
</Panel>
```

## snippet 属性

专门给 PUI 定制的属性，自动载入 snippet，`<Panel snippet="MyBtton" />`

## vars 和 dialogVariables

两者是一样的，`dialogVariables` 是为了兼容[ark120202/react-panorama](https://github.com/ark120202/react-panorama)

-   当值为`string`时，调用`SetDialogVariable`，如果以`#`开头则调用`SetDialogVariableLocString`
-   当值为`number`时，调用`SetDialogVariableInt`
-   当值为`Date`时，调用`SetDialogVariableTime`

针对 Label 做了一些调整，vars 和 dialogVariablesh 会先写入，然后再写入`Label.text`, 如果 text 以`#`开头会调用`$.Localize(text, Label)`。

```jsx
<Label vars={{ name: 'X.X' }} text="#name_of_x" />
```
