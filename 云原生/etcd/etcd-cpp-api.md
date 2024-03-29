## etcd::Response

```
action()							//return string 有以下字符串[create] [set] [delete]
events()							//return vector 返回事件Event  
	Event:
		kv()						//返回本次发生变化的key的修改后的value
		prev_kv()					//返回本次发生变化的key的修改前的value
		

```

## Usage

### get

```c++
  etcd::Client etcd("http://127.0.0.1:2379");
  etcd::Response response = etcd.get("/test/key1").get();
  std::cout << response.value().as_string();
```

​		or

```c++
  etcd::Client etcd("http://127.0.0.1:2379");
  pplx::task<etcd::Response> response_task = etcd.get("/test/key1");
  // ... do something else
  etcd::Response response = response_task.get();
  std::cout << response.value().as_string();
```

​		or

```c++
  etcd::Client etcd("http://127.0.0.1:2379");
  etcd.get("/test/key1").then([](etcd::Response response)
  {
    std::cout << response.value().as_string();
  });

  // ... your code can continue here without any delay
```

​		or

```c++
  etcd::Client etcd("http://127.0.0.1:2379");
  etcd.get("/test/key1").then([](pplx::task<etcd::Response> response_task)
  {
    try
    {
      etcd::Response response = response_task.get(); // can throw
      std::cout << response.value().as_string();
    }
    catch (std::exception const & ex)
    {
      std::cerr << ex.what();
    }
  });
```

##### key is not exist

error_code  = 100

### put操作

```c++
//租约
etcd::Client client(etcd_addr);
auto leaseRes = client.leasegrant(30);
//使用wait等待操作完成，不然可能在返回之前没有完成操作
client.put("/my/test/key1","1",leaseRes.get().value().lease()).wait();
client.put("/my/test/key2","2",leaseRes.get().value().lease()).wait();
```

### 前缀遍历和前缀删除

```c++
client.ls("/my/test").then([](etcd::Response resp){
    auto keys = resp.keys();
    for(int i = 0 ; i < keys.size() ; i ++){
        std::cout << keys[i] << " " << resp.values().at(i).as_string() << '\n';
    }
});
client.rmdir("/my/test/key",true).wait();
client.ls("/my/test").then([](etcd::Response resp){
    auto keys = resp.keys();
    for(int i = 0 ; i < keys.size() ; i ++){
        std::cout << keys[i] << " " << resp.values().at(i).as_string() << '\n';
    }
});
```

### 事务

```c++
etcdv3::Transaction txn;
etcdv3::CompareResult result = etcdv3::CompareResult::EQUAL;
//比较最近一次修改的版本号
txn.add_compare_mod("/my/test/key1",result,0);
txn.add_success_put("/my/test/key4","success");
txn.add_failure_put("/my/test/key4","fail");
//比较key被修改的次数
txn.add_compare_version("/my/test2",0);
//比较key创建时全局版本号
txn.add_compare_create("/my/test2",result,0);
//比较key的value
txn.add_compare_value("/my/test/key1",result,"value");
//比较key的租约id，没有租约则失败
txn.add_compare_lease("/my/test/key1",result,leaseRes.get().value().lease());

txn.add_success_put("/my/test/key5","success");
txn.add_failure_put("/my/test/key5","fail");
client.txn(txn).then([](etcd::Response resp){
    std::cout << resp.value().as_string() <<'\n';
});
```

### 锁

```
client.lock("/api/login/mutex",10).wait();		//没有unlock自动续租，租约防止断开连接锁不能释放,一定要wait()阻塞住，不然非阻塞会上锁失败也会继续
client.unlock("/api/login/mutex").wait();
```

### 分布式选主

```c++
client.leasekeepalive(20).then([&](etcd::Response resp){
std::thread observer_th([&](){
    std::cout << "观察者监视线程启动\n";
    auto observer = client.observe("campaign");
    observer->WaitOnce();
    std::cout << "observe " << resp.value().key() << " as the leader: " << resp.value().as_string() << std::endl;

	});
std::cout << "开始竞选\n";
client.campaign("campaign",resp.value().lease(),"XXXX").wait();
std::this_thread::sleep_for(std::chrono::seconds(30));
client.resign("campaign",resp.value().lease(),resp.value().key(),resp.value().created_index());
std::cout << "卸任\n";
observer_th.join();
});
```

### Keepalive

返回的keepalive对象被销毁lease到期就无法续租

```c++
class Etcd_Regist{
public:
    using uptr = std::unique_ptr<Etcd_Regist>;
    void regist(std::string Scheme,std::string ServiceName,std::string ServiceAddr){
        m_keepalive = client.leasekeepalive(10).get();
        int64_t leaseid = m_keepalive->Lease();
        std::string key = "/";
        key = key + Scheme + '/' + ServiceName + '/' + ServiceAddr;
        client.put(key,ServiceAddr,leaseid).wait();
        std::cout << client.get(key).get().value().as_string() <<'\n';
        printf("%lx\n",leaseid);
    }
private:
    std::shared_ptr<etcd::KeepAlive> m_keepalive;
};
```

### Watcher

参数 recursive 为true时监视前缀，false监视相同的key值

```
 std::string etcd_addr = "http://127.0.0.1:2379,http://127.0.0.1:12379,http://127.0.0.1:22379";
 etcd::Watcher watcher(etcd_addr,"/api/regist",Watcher_cb,true);
```

