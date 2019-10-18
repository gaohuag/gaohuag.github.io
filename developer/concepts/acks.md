# 确认

在具有最多一次语义的系统中，有时会丢失消息。如果您的应用程序正在执行请求-应答，那么它应该使用超时来处理任何网络或应用程序故障。
在请求上设置超时并使用处理超时的代码总是一个好主意。在发布事件或数据流时，确保消息送达的一种方法是将其转换为带有确认消息(ACKs)概念的请求-应答。
在NATS中，ACK可以是一个空消息，一个没有有效负载的消息。

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
    sub1[shape=box, style="rounded", label="Subscriber"];
  }

  publisher -> subject [label="msg1"];
  publisher -> reply [style="invis", weight=2];
  subject -> sub1 [label="msg1"];
  sub1 -> reply [label="ack"];
  reply -> publisher;
}
</code></div>

因为 ACK 可以是空的，所以占用非常小的网络带宽,但ACK的想法由简单的 fire-and-forget 变成fire-and-know世界,
发送方可以确定消息被收到,浏览[scatter-gather pattern](reqreply.md)了解其他几个方面。