# 队列订阅者和可伸缩性

NATS提供了一个称为分布式队列的内置负载均衡特性。使用队列订阅者将在一组订阅者之间均衡消息传递，这些订阅者可用于提供应用程序容错和大规模工作负载处理。

要创建队列订阅，订阅者需要注册队列名称。具有相同队列名称的所有订阅者组成队列组。这不需要配置。
当注册主题上的消息发布时，将随机选择组中的一个成员来接收消息。尽管队列组有多个订阅者，但每个消息仅被一个订阅者使用。

NATS的一个重要特性是队列组是由应用程序及其队列订阅者定义的，而不是在服务器配置上定义的。
Queue subscribers are ideal for scaling services. Scale up is as simple as running another application, 
scale down is terminating the application with a signal that drains the in flight requests.
队列订阅者是扩展服务的理想对象。扩容与运行另一个应用程序一样简单，缩放是使用一个信号终止应用程序，该信号将处理完正在运行的请求后执行。

这种灵活性和缺少任何配置更改使NATS成为一种优秀的服务通信技术，可以与所有平台技术一起工作。



<div class="graphviz"><code data-viz="dot">
digraph nats_queues {
  rankdir=LR
  publisher [shape=box, style="rounded", label="Publisher"];
  subject [shape=circle, label="Queue"];
  sub1 [shape=box, style="rounded", label="Subscriber"];
  sub2 [shape=box, style="rounded", label="Subscriber"];
  sub3 [shape=box, style="rounded", label="Subscriber"];

  publisher -> subject [label="msgs 1,2,3"];
  subject -> sub1 [label="msg 2"];
  subject -> sub2 [label="msg 1"];
  subject -> sub3 [label="msg 3"];
}
</code></div>

尝试自己的NATS队列订阅，使用一个在线的服务器，参考[queueing tutorial](../tutorials/queues.md)。
