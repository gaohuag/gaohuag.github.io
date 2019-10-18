# 发布-订阅

NATS实现了一对多通信的发布-订阅 消息分发模型。发布者发送关于主题的消息，而监听该主题的任何活动订阅者将接收该消息。
订阅者还可以注册对通配符主题感兴趣的内容，通配符的工作原理有点像正则表达式(但只有一点点)。这种一对多的模式有时被称为扇出。


<div class="graphviz"><code data-viz="dot">
digraph nats_pub_sub {
  rankdir=LR
  publisher [shape=box, style="rounded", label="Publisher"];
  subject [shape=circle, label="Subject"];
  sub1 [shape=box, style="rounded", label="Subscriber"];
  sub2 [shape=box, style="rounded", label="Subscriber"];
  sub3 [shape=box, style="rounded", label="Subscriber"];

  publisher -> subject [label="msg1"];
  subject -> sub1 [label="msg1"];
  subject -> sub2 [label="msg1"];
  subject -> sub3 [label="msg1"];
}
</code></div>

尝试使用启动好的服务器自己发布订阅主题，请浏览[pub-sub tutorial](../tutorials/pubsub.md)。