# 序列号

一对多消息的一个常见问题是，由于网络故障，消息可能丢失或丢失。解决这种情况的一个简单模式是在消息中包含一个序列id。
接收方可以检查序列id，看看是否遗漏了什么。

在缺乏新数据的情况下，序列号与心跳结合在一起，形成一个强大而有弹性的模式来检测丢失。
存储和持久化消息的系统也可以解决这个问题，但有时对于手边的问题来说，这样做有些过头了，通常会导致额外的管理和运维成本。

<div class="graphviz"><code data-viz="dot">
digraph nats_pub_sub {
  rankdir=LR
  publisher [shape=box, style="rounded", label="Publisher"];
  subject [shape=circle, label="Subject"];
  sub [shape=box, style="rounded", label="Subscriber"];

  publisher -> subject [label="updates.1"];
  publisher -> subject [label="updates.2"];
  publisher -> subject [label="updates.3"];

  subject -> sub [label="updates.*"];
}
</code></div>

为了真正的利用好序列id，需要记住以下几点:
* 每个发送者必须使用他们自己的序列
* 如有可能，接收者应能以id查询丢失的消息


使用NATS，您可以在消息中嵌入序列id，或者将它们作为标记包含在主题中。例如，发送者可以向 `updates.1`, `updates.2`发送消息等……
订阅者订阅`updates.*`并解析主题以确定序列id。

可能payload为空，所以可以将序列号放置到主题中，不要放在payload中。