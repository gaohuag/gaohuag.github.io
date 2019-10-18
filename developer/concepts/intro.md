# 什么是NATS

NATS消息传递使得计算机应用程序和服务把交互的数据隔离出来。
消息解决的是主题问题，不依赖于网络位置。
这提供了应用程序或服务与底层物理网络之间的抽象层。
数据被编码并作为消息打包，由发布者发送。
消息由一个或多个订阅者接收、解码和处理。


NATS使程序可以很容易地跨不同的环境、语言、云提供商和内部系统进行通信。客户机通常通过单个URL连接到NATS系统，然后向主题订阅或发布消息。
通过这种简单的设计，NATS允许程序共享通用的消息处理代码，隔离资源和相互依赖，并通过轻松处理增加的消息量(无论是服务请求还是流数据)来扩展。
<div class="graphviz"><code data-viz="dot">
graph nats {
  graph [splines=ortho, nodesep=1];

  publisher [shape="record", label="{Application 1 | <nats> NATS Publisher}"];
  application [shape="record", label="{Application 3 | <nats>  }"];
  natsserver [shape="box", label="", width=4, height=0, penwidth=1];
  subscriber [shape="record", label="{<nats> NATS Subscriber | Application 2}"];

  publisher:nats -- natsserver [penwidth=2];
  application:nats -- natsserver;
  natsserver -- subscriber:nats [penwidth=2, dir="forward"];
}
</code></div>

NATS core提供**最多一次**的服务质量。如果订阅者没有监听主题(没有主题匹配)，或者在发送消息时未激活，则不会接收消息。
这与TCP/IP提供的保证级别相同。默认情况下，NATS是一个“发了就忘”的消息传递系统。如果你需要更高水平的服务，
你可以使用[NATS流](/nats_streaming/intro.md)，或者在你的客户端应用程序中增加可靠性，提供可靠的、可扩展的参考设计。