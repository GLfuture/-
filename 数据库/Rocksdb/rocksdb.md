[RocksDB-VIP-2304.pdf](file:///D:/零声Linux/rocksdb/RocksDB-VIP-2304.pdf)

编译:

```shell
g++ a.cc -o a -std=c++17 librocksdb.a  -I/usr/local/rocksdb  -lpthread -ldl -lrt -lsnappy -lgflags -lz -lbz2 -llz4 -lzstd
```

