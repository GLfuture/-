[skynet组件以及开发-2207-VIP.pdf](file:///D:/零声Linux/skynet/skynet组件以及开发-2207-VIP.pdf)

### skynet

```
ln -s ../src/skynet/ ./skynet    //创建软链接
```

一个lua文件就是一个actor

```lua
local skynet = require "skynet"
//所有actor的入口函数,actor的初始化
skynet.start(function()
	skynet.dispatch("lua",function(session,source,cmd,...)
             //接收actor的消息，第一个参数(消息类型),function中的session为会话(唤醒协程)，source(消息从哪来)，cmd(命令)
            
     end)      
end)
```

