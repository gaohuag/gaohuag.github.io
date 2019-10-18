# 基于主题的消息传递

从根本上说，NATS是关于发布和监听消息的。这两者都严重依赖于_主题_相关的消息流或主题。
简单地说，subject就是一串字符，发布者和订阅者可以用它来查找彼此。

<div class="graphviz"><code data-viz="dot">
digraph g {
  rankdir=LR
  publisher [shape=box, style="rounded", label="PUB time.us"];
  subject [shape=circle, fixedsize="true", width="1.0", height="1.0", label="nats-server"];
  sub1 [shape=box, style="rounded", label="SUB time.us"];
  sub2 [shape=box, style="rounded", label="SUB time.us"];

  publisher -> subject [label="msg"];
  subject -> sub1 [label="msg"];
  subject -> sub2 [label="msg"];
}
</code></div>

NATS服务器保留了一些特殊字符，规范中说，只有“字母-数字”字符加上“.”应该用于主题名。主题是大小写敏感的，不能包含空格。
为了跨客户机的安全，应该使用ASCII字符，尽管将来可能会发生变化。

## 主题层次结构

`.`字符用于创建主题层次结构。例如，一个世界时钟应用程序可能定义以下内容来逻辑地分组相关的主题:
```markup
time.us
time.us.east
time.us.east.atlanta
time.eu.east
time.eu.warsaw
```

## 通配符


NATS提供了两个 _通配符_ ，可以代替点分隔主题中的一个或多个元素。
订阅者可以使用这些通配符侦听多个主题，但发布者将始终使用完全指定的主题，而不使用通配符。


### 匹配单个 Token

 which would match `time.us.east` and `time.eu.east`.
第一个通配符是 `*`，它将匹配单个token。例如，如果一个应用程序想要监听东部时区，他们可以订阅 `time.*.east`。
匹配`time.us.east` 和 `time.eu.east`。

<div class="graphviz"><code data-viz="dot">
digraph g {
  rankdir=LR
  publisher [shape=box, style="rounded", label="PUB time.us.east"];
  subject [shape=circle, fixedsize="true", width="1.0", height="1.0", label="nats-server"];
  sub1 [shape=box, style="rounded", label="SUB time.*.east"];
  sub2 [shape=box, style="rounded", label="SUB time.us.east"];

  publisher -> subject [label="msg"];
  subject -> sub1 [label="msg"];
  subject -> sub2 [label="msg"];
}
</code></div>

### 匹配多个令牌

第二个通配符是 `>` ，它将匹配一个或多个标记，并且只能出现在主题的末尾。
例如,“`time.us.>`将匹配 `time.us.east` and `time.us.east.atlanta`,而`time.us.*` 只配得上`time.us.east` 。因为它只能匹配一个符号。

<div class="graphviz"><code data-viz="dot">
digraph g {
  rankdir=LR
  publisher [shape=box, style="rounded", label="PUB time.us.east.atlanta"];
  subject [shape=circle, fixedsize="true", width="1.0", height="1.0", label="nats-server"];
  sub1 [shape=box, style="rounded", label="SUB time.us.east.atlanta"];
  sub2 [shape=box, style="rounded", label="SUB time.us.*"];
  sub3 [shape=box, style="rounded", label="SUB time.us.>"];

  publisher -> subject [label="msg"];
  subject -> sub2 [style="invis"];
  subject -> sub1 [label="msg"];
  subject -> sub3 [label="msg"];
}
</code></div>

### 监控和窃听


根据您的安全配置，通配符可以通过创建有时称为 *wire tap* 的东西来用于监控。在最简单的情况下，您可以用通配符 `>`创建一个订阅服务器。
这个应用程序将接收所有在NATS集群上发送的消息(同样，取决于安全设置)。
