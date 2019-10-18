# 请求-应答

请求-应答是现代分布式系统中的一种常见模式。发送请求时，应用程序要么指定超时时间等待响应，要么异步接收响应。

现代系统日益增加的复杂性要求具有诸如位置透明性、伸缩性、可监控等特征。许多技术需要额外的组件、边站和代理来完成完整的特性集。

NATS通过其核心通信机制(发布和订阅)支持这种模式。请求在给定的主题上与应答主题一起发布，应答者侦听该主题并将响应发送到应答主题。回复主题

通常是一个名为\_INBOX的主题，该主题将被动态地返回给请求者，而不考虑任何一方的位置。

NATS允许多个响应器运行并形成动态队列组，以实现透明的扩展。NATS应用程序在退出之前耗尽的能力允许按比例缩小，而不会丢失任何请求。
由于NATS是基于发布-订阅的，所以可观察性与运行另一个应用程序一样简单，该应用程序可以查看请求和响应以度量延迟、监控异常、直接可伸缩性等。

NATS的能力甚至允许多个响应，其中第一个响应被利用，而系统有效地丢弃了附加的响应。这允许一个复杂的模式有多个响应器减少响应延迟和抖动。
<div class="graphviz"><code data-viz="dot">
digraph nats_request_reply {
  rankdir=LR

  subgraph {
    publisher [shape=box, style="rounded", label="Publisher"];
  }

  subgraph {
    subject [shape=circle, label="Subject"];
    reply [shape=circle, label="Reply"];
    {rank = same subject reply}
  }

  subgraph {
    sub1 [shape=box, style="rounded", label="Subscriber"];
    sub2 [shape=box, style="rounded", label="Subscriber"];
    sub3 [shape=box, style="rounded", label="Subscriber"];
  }

  publisher -> subject [label="msg1"];
  publisher -> reply [style="invis", weight=2];
  reply -> sub3 [style="invis", weight=2];
  subject -> sub1 [label="msg1", style="dotted"];
  subject -> sub2 [label="msg1", style="dotted"];
  subject -> sub3 [label="msg1"];
  sub3 -> reply;
  reply -> publisher;
}
</code></div>

自己尝试一下NATS的请求回复，参考 [request/reply tutorial](../tutorials/reqreply.md)，使用启动好的服务器做测试。
