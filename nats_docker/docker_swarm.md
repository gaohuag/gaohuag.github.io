## Docker Swarm

#### Step 1:
为集群创建一个覆盖网络(在本例中为`nats-cluster-example`)，并实例化一个初始的NATS服务器。

首先创建一个覆盖网络:


```sh
% docker network create --driver overlay nats-cluster-example
```

接下来，为NATS集群实例化一个初始的“种子”服务器，监听其他服务器在6222端口上加入到它的路由:

```sh
% docker service create --network nats-cluster-example --name nats-cluster-node-1 nats:1.0.0 -cluster nats://0.0.0.0:6222 -DV
```

#### Step 2:
第二步是创建另一个连接到覆盖网络内的NATS服务器的服务。注意，我们在`nats-cluster-node-1`连接到服务器:

```sh
% docker service create --name ruby-nats --network nats-cluster-example wallyqs/ruby-nats:ruby-2.3.1-nats-v0.8.0 -e '
  NATS.on_error do |e|
    puts "ERROR: #{e}"
  end
  NATS.start(:servers => ["nats://nats-cluster-node-1:4222"]) do |nc|
    inbox = NATS.create_inbox
    puts "[#{Time.now}] Connected to NATS at #{nc.connected_server}, inbox: #{inbox}"

    nc.subscribe(inbox) do |msg, reply|
      puts "[#{Time.now}] Received reply - #{msg}"
    end

    nc.subscribe("hello") do |msg, reply|
      next if reply == inbox
      puts "[#{Time.now}] Received greeting - #{msg} - #{reply}"
      nc.publish(reply, "world")
    end

    EM.add_periodic_timer(1) do
      puts "[#{Time.now}] Saying hi (servers in pool: #{nc.server_pool}"
      nc.publish("hello", "hi", inbox)
    end
  end'
```

#### Step 3:
现在你可以通过更多的docker服务向集群中添加更多的节点，在`-routes`参数中引用种子服务器:
```sh
% docker service create --network nats-cluster-example --name nats-cluster-node-2 nats:1.0.0 -cluster nats://0.0.0.0:6222 -routes nats://nats-cluster-node-1:6222 -DV
```

在本例中，`nats-cluster-node-1`通过自动发现特性来播种集群的其余部分。现在，NATS服务器`nat-cluster-node-1`和`nat-cluster-node-2`集群在一起。


添加更多的订阅者副本:

```sh
% docker service scale ruby-nats=3
```

然后确定Docker集群的分布:

```sh
% docker service ps ruby-nats
ID                         NAME         IMAGE                                     NODE    DESIRED STATE  CURRENT STATE          ERROR
25skxso8honyhuznu15e4989m  ruby-nats.1  wallyqs/ruby-nats:ruby-2.3.1-nats-v0.8.0  node-1  Running        Running 2 minutes ago  
0017lut0u3wj153yvp0uxr8yo  ruby-nats.2  wallyqs/ruby-nats:ruby-2.3.1-nats-v0.8.0  node-1  Running        Running 2 minutes ago  
2sxl8rw6vm99x622efbdmkb96  ruby-nats.3  wallyqs/ruby-nats:ruby-2.3.1-nats-v0.8.0  node-2  Running        Running 2 minutes ago
```

下面是向集群添加更多的NATS服务器节点后的示例输出——请注意，客户机通过自动发现功能动态地知道更多的节点是集群的一部分!
```sh
[2016-08-15 12:51:52 +0000] Saying hi (servers in pool: [{:uri=>#<URI::Generic nats://10.0.1.3:4222>, :was_connected=>true, :reconnect_attempts=>0}]
[2016-08-15 12:51:53 +0000] Saying hi (servers in pool: [{:uri=>#<URI::Generic nats://10.0.1.3:4222>, :was_connected=>true, :reconnect_attempts=>0}]
[2016-08-15 12:51:54 +0000] Saying hi (servers in pool: [{:uri=>#<URI::Generic nats://10.0.1.3:4222>, :was_connected=>true, :reconnect_attempts=>0}]
[2016-08-15 12:51:55 +0000] Saying hi (servers in pool: [{:uri=>#<URI::Generic nats://10.0.1.3:4222>, :was_connected=>true, :reconnect_attempts=>0}, {:uri=>#<URI::Generic nats://10.0.1.7:4222>, :reconnect_attempts=>0}, {:uri=>#<URI::Generic nats://10.0.1.6:4222>, :reconnect_attempts=>0}]
```

增加更多可以回复的工人后的样本输出(因为忽略了自己的回复):

```sh
[2016-08-15 16:06:26 +0000] Received reply - world
[2016-08-15 16:06:26 +0000] Received reply - world
[2016-08-15 16:06:27 +0000] Received greeting - hi - _INBOX.b8d8c01753d78e562e4dc561f1
[2016-08-15 16:06:27 +0000] Received greeting - hi - _INBOX.4c35d18701979f8c8ed7e5f6ea
```

### 等等...

从这里开始，您可以通过简单地添加具有新服务名称的服务器来试验添加到NATS集群，这将路由到种子服务器`nat-cluster-node-1`。正如上面所看到的，客户机将自动更新，以了解集群中有新的服务器可用。

```sh
% docker service create --network nats-cluster-example --name nats-cluster-node-3 nats:1.0.0 -cluster nats://0.0.0.0:6222 -routes nats://nats-cluster-node-1:6222 -DV
```