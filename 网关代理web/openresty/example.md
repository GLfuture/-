## openresty实现黑名单

下面方法实际都不可取，真正可取的方法是现在共享内存中加载第一次黑名单，之后定义一个server把一个端口暴露给内部，然后定义location针对不同的url对共享内存进行添加黑名单，删除黑名单等操作

### conf1

```
worker_processes 8;
events{
    worker_connections 10240;
}

http{
    server{
        listen 8080;
        location / {
            # init_by_lua_block{
            #     #加载一些耗时模块或者初始化一些全局变量
            #     require "resty.redis"
            # }
            rewrite_by_lua_block{
                local args=ngx.req.get_uri_args()
                if args["jump"]=="1" then
                    return ngx.redirect("http://baidu.com")
                elseif args["jump"]=="2" then
                    return ngx.redirect("/jump_here")
                end
            }
            content_by_lua_block{
                ngx.say("hello","\t",ngx.var.remote_addr)
            }
        }
        location /jump_here{
            content_by_lua_block{
                ngx.say("jump_here hello","\t",ngx.var.remote_addr)
            }
            #修改应答消息体
            body_filter_by_lua_block{
            -- 拿到消息体
                local chunk = ngx.arg[1]
                ngx.arg[1] = chunk:gsub("hello","lyj")
            }
        }
    }
}
```

### black_v1.lua

```
local redis = require "resty.redis"
local red = redis:new()
local ok , err = red:connect("8.130.92.28",6379)
if not ok then
    return ngx.exit(301)
end

local res,err=red:auth("123456")
if not res then
    ngx.log(ngx.ERR, "failed to authenticate Redis connection: ", err)
    return
end

local ip=ngx.var.remote_addr

local exists , err=red:sismember("black_list",ip)
if exists == 1 then
    return ngx.exit(403)
end
```

### conf2

```
worker_processes 8;
events{
    worker_connections 10240;
}

http{
    #创建共享内存		共享内存名字	共享内存大小
	lua_shared_dict bklist	1m;
    init_worker_by_lua_file ./app/init_worker.lua;
    server{
        listen 8080;
        server_name localhost;
        location /black_v1 {
            access_by_lua_file ./app/black_v1.lua;
        }
        location /black_v2 {
            access_by_lua_file ./app/black_v2.lua;
            content_by_lua_block{
                ngx.say(ngx.var.remote_addr)
            }
        }
    }
}

stream{
    upstream ups{
        server 127.0.0.1:8080;
    }
    server{
        listen 9999;
        proxy_pass ups;
        #由于转发会丢失客户端ip，所以需要用到转发协议，默认v1(ASCII码)形式，还有v2版本（字节流形式）
        #proxy_protocol on;
    }
}
```

### init_worker.lua

```

if ngx.worker.id() ~= 0 then
    return
end

local redis = require "resty.redis"
local bklist = ngx.shared.bklist

local function update_blacklist()
    local red = redis:new()
    local ok, err = red:connect("8.130.92.28", 6379)
    if not ok then
        return
    end
    local res,err=red:auth("123456")
    if not res then
        return 
    end
    local black_list,err = red:smembers("black_list")
    bklist:flush_all()
    for _, v in pairs(black_list) do
        bklist:set(v, true)
    end
    ngx.timer.at(5, update_blacklist)
end

ngx.timer.at(5, update_blacklist)
```

### black_v2.lua

```
local bklist = ngx.shared.bklist
local ip = ngx.var.remote_addr

if bklist:get(ip) then
    return ngx.exit(403)
end
```

