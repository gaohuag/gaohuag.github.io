## Windows Service

NATS服务器支持作为Windows服务运行。实际上，这是在Windows上运行NATS的推荐方法。目前没有安装程序，用户应该使用 `sc.exe` 来安装服务。
```batch
sc.exe create nats-server binPath= "%NATS_PATH%\nats-server.exe [nats-server flags]"
sc.exe start nats-server
```
上面将创建并启动一个 `nats-server` 服务。注意，应该在创建服务时提供 nats-server 标志。这允许在单个Windows服务器上运行多个NATS服务器配置，
每个安装的NATS服务器服务使用一个1:1的服务实例。一旦服务开始运行，就可以使用 `sc.exe` 或者 `nats-server.exe --signal`进行控制：

```batch
REM Reload server configuration
nats-server.exe --signal reload

REM Reopen log file for log rotation
nats-server.exe --signal reopen

REM Stop the server
nats-server.exe --signal stop
```
上述命令将默认用于控制 `nats-server` 服务。如果服务是另一个名称，可以指定:
```batch
nats-server.exe --signal stop=<service name>
```

有关信号的完整列表，请参见 [process signaling](/nats_admin/signals.md).