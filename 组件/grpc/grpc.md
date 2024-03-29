## grpc

前置：D:\Typora\MarkDown\C++网络编程\自定义协议.md

### 安装

[1-1-gRPC C++开发环境搭建](file:///D:/零声Linux/grpc/1-1-gRPC C++开发环境搭建.html)

### 优点

[1-2-gRPC远程调用v1.2.pdf](file:///D:/零声Linux/grpc/1-2-gRPC远程调用v1.2.pdf)

### 实践

[1-3-grpc for c++使用案例-手把手.pdf](file:///D:/零声Linux/grpc/1-3-grpc-手把手案例补充/1-3-grpc-手把手案例补充/1-3-grpc for c++使用案例-手把手.pdf)

[2-gRPC网络线程模型和应用实践.pdf](file:///D:/零声Linux/grpc/2-gRPC网络线程模型和应用实践.pdf)

### 小知识

package IM.Login中的.会变成::，即生成IM的命名空间

### 问题

如果项目中需要使用openssl，由于grpc默认使用boringssl需要

```
sudo apt-get install libssl-dev
```

```
cmake -DgRPC_SSL_PROVIDER=OpenSSL . && make -j4 && make install
```

### 生成

```
protoc -I . --cpp_out=. --grpc_out=. --plugin=protoc-gen-grpc=`which grpc_cpp_plugin` *.proto
```



### 代码

##### CMakeLists

```cmake
cmake_minimum_required(VERSION 3.5.1)

project(IMLogin C CXX)

include(../cmake/common.cmake)
#支持Debug模式
SET(CMAKE_BUILD_TYPE "Debug")
# Proto file
get_filename_component(im_proto "./IMlogin.proto" ABSOLUTE)
get_filename_component(im_proto_path "${im_proto}" PATH)

# Generated sources
set(im_proto_srcs "${CMAKE_CURRENT_BINARY_DIR}/IMlogin.pb.cc")
set(im_proto_hdrs "${CMAKE_CURRENT_BINARY_DIR}/IMlogin.pb.h")
set(im_grpc_srcs "${CMAKE_CURRENT_BINARY_DIR}/IMlogin.grpc.pb.cc")
set(im_grpc_hdrs "${CMAKE_CURRENT_BINARY_DIR}/IMlogin.grpc.pb.h")
# 根据proto文件生成xx.cc xx.h
add_custom_command(
      OUTPUT "${im_proto_srcs}" "${im_proto_hdrs}" "${im_grpc_srcs}" "${im_grpc_hdrs}"
      COMMAND ${_PROTOBUF_PROTOC}
      ARGS --grpc_out "${CMAKE_CURRENT_BINARY_DIR}"
        --cpp_out "${CMAKE_CURRENT_BINARY_DIR}"
        -I "${im_proto_path}"
        --plugin=protoc-gen-grpc="${_GRPC_CPP_PLUGIN_EXECUTABLE}"
        "${im_proto}"
      DEPENDS "${im_proto}")

# Include generated *.pb.h files
include_directories("${CMAKE_CURRENT_BINARY_DIR}")

# im_grpc_proto
add_library(im_grpc_proto
  ${im_grpc_srcs}
  ${im_grpc_hdrs}
  ${im_proto_srcs}
  ${im_proto_hdrs})
target_link_libraries(im_grpc_proto
  ${_REFLECTION}
  ${_GRPC_GRPCPP}
  ${_PROTOBUF_LIBPROTOBUF})

# Targets greeter_[async_](client|server)
#需要替换 _target
foreach(_target
  client server
  )
  add_executable(${_target} "${_target}.cc")
  target_link_libraries(${_target}
    im_grpc_proto
    ${_REFLECTION}
    ${_GRPC_GRPCPP}
    ${_PROTOBUF_LIBPROTOBUF})
endforeach()

```

##### cmake/common.cmake

```cmake
# Copyright 2018 gRPC authors.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#
# cmake build file for C++ route_guide example.
# Assumes protobuf and gRPC have been installed using cmake.
# See cmake_externalproject/CMakeLists.txt for all-in-one cmake build
# that automatically builds all the dependencies before building route_guide.

cmake_minimum_required(VERSION 3.5.1)

set (CMAKE_CXX_STANDARD 11)

if(MSVC)
  add_definitions(-D_WIN32_WINNT=0x600)
endif()

find_package(Threads REQUIRED)

if(GRPC_AS_SUBMODULE)
  # One way to build a projects that uses gRPC is to just include the
  # entire gRPC project tree via "add_subdirectory".
  # This approach is very simple to use, but the are some potential
  # disadvantages:
  # * it includes gRPC's CMakeLists.txt directly into your build script
  #   without and that can make gRPC's internal setting interfere with your
  #   own build.
  # * depending on what's installed on your system, the contents of submodules
  #   in gRPC's third_party/* might need to be available (and there might be
  #   additional prerequisites required to build them). Consider using
  #   the gRPC_*_PROVIDER options to fine-tune the expected behavior.
  #
  # A more robust approach to add dependency on gRPC is using
  # cmake's ExternalProject_Add (see cmake_externalproject/CMakeLists.txt).

  # Include the gRPC's cmake build (normally grpc source code would live
  # in a git submodule called "third_party/grpc", but this example lives in
  # the same repository as gRPC sources, so we just look a few directories up)
  add_subdirectory(../../.. ${CMAKE_CURRENT_BINARY_DIR}/grpc EXCLUDE_FROM_ALL)
  message(STATUS "Using gRPC via add_subdirectory.")

  # After using add_subdirectory, we can now use the grpc targets directly from
  # this build.
  set(_PROTOBUF_LIBPROTOBUF libprotobuf)
  set(_REFLECTION grpc++_reflection)
  if(CMAKE_CROSSCOMPILING)
    find_program(_PROTOBUF_PROTOC protoc)
  else()
    set(_PROTOBUF_PROTOC $<TARGET_FILE:protobuf::protoc>)
  endif()
  set(_GRPC_GRPCPP grpc++)
  if(CMAKE_CROSSCOMPILING)
    find_program(_GRPC_CPP_PLUGIN_EXECUTABLE grpc_cpp_plugin)
  else()
    set(_GRPC_CPP_PLUGIN_EXECUTABLE $<TARGET_FILE:grpc_cpp_plugin>)
  endif()
elseif(GRPC_FETCHCONTENT)
  # Another way is to use CMake's FetchContent module to clone gRPC at
  # configure time. This makes gRPC's source code available to your project,
  # similar to a git submodule.
  message(STATUS "Using gRPC via add_subdirectory (FetchContent).")
  include(FetchContent)
  FetchContent_Declare(
    grpc
    GIT_REPOSITORY https://github.com/grpc/grpc.git
    # when using gRPC, you will actually set this to an existing tag, such as
    # v1.25.0, v1.26.0 etc..
    # For the purpose of testing, we override the tag used to the commit
    # that's currently under test.
    GIT_TAG        vGRPC_TAG_VERSION_OF_YOUR_CHOICE)
  FetchContent_MakeAvailable(grpc)

  # Since FetchContent uses add_subdirectory under the hood, we can use
  # the grpc targets directly from this build.
  set(_PROTOBUF_LIBPROTOBUF libprotobuf)
  set(_REFLECTION grpc++_reflection)
  set(_PROTOBUF_PROTOC $<TARGET_FILE:protoc>)
  set(_GRPC_GRPCPP grpc++)
  if(CMAKE_CROSSCOMPILING)
    find_program(_GRPC_CPP_PLUGIN_EXECUTABLE grpc_cpp_plugin)
  else()
    set(_GRPC_CPP_PLUGIN_EXECUTABLE $<TARGET_FILE:grpc_cpp_plugin>)
  endif()
else()
  # This branch assumes that gRPC and all its dependencies are already installed
  # on this system, so they can be located by find_package().

  # Find Protobuf installation
  # Looks for protobuf-config.cmake file installed by Protobuf's cmake installation.
  set(protobuf_MODULE_COMPATIBLE TRUE)
  find_package(Protobuf CONFIG REQUIRED)
  message(STATUS "Using protobuf ${Protobuf_VERSION}")

  set(_PROTOBUF_LIBPROTOBUF protobuf::libprotobuf)
  set(_REFLECTION gRPC::grpc++_reflection)
  if(CMAKE_CROSSCOMPILING)
    find_program(_PROTOBUF_PROTOC protoc)
  else()
    set(_PROTOBUF_PROTOC $<TARGET_FILE:protobuf::protoc>)
  endif()

  # Find gRPC installation
  # Looks for gRPCConfig.cmake file installed by gRPC's cmake installation.
  find_package(gRPC CONFIG REQUIRED)
  message(STATUS "Using gRPC ${gRPC_VERSION}")

  set(_GRPC_GRPCPP gRPC::grpc++)
  if(CMAKE_CROSSCOMPILING)
    find_program(_GRPC_CPP_PLUGIN_EXECUTABLE grpc_cpp_plugin)
  else()
    set(_GRPC_CPP_PLUGIN_EXECUTABLE $<TARGET_FILE:gRPC::grpc_cpp_plugin>)
  endif()
endif()
```

### 同步

##### server

```c++
#include<iostream>
#include<string>
#include<memory>
//grpc头文件
#include<grpcpp/ext/proto_server_reflection_plugin.h>
#include<grpcpp/grpcpp.h>
#include<grpcpp/health_check_service_interface.h>

#include"IMlogin.grpc.pb.h"

using std::string;
//命名空间Server
using grpc::Server;
using grpc::ServerBuilder;
using grpc::ServerContext;
using grpc::Status;
using IM::Login::IMlogin;
using IM::Login::IMLoginReq;
using IM::Login::IMLoginRes;
using IM::Login::IMRegistReq;
using IM::Login::IMRegistRes;
//继承Service实现rpc
class IMLoginServicerImpl:public IMlogin::Service
{
    //注册
    virtual Status Regist(ServerContext* context, const IMRegistReq* request, IMRegistRes* response) override{
        std::cout<<"Regist "<< request->user_name()<<std::endl;
        response->set_user_name(request->user_name());
        response->set_user_id(10);
        response->set_result_code(0);
        return Status::OK;
    }
    //登录
    virtual Status Login(ServerContext* context, const IMLoginReq* request, IMLoginRes* response) override{
        std::cout<<"Login "<< request->user_name()<<std::endl;
        response->set_user_id(10);
        response->set_result_code(0);
        return Status::OK;
    }
};

void RunServer()
{
    string server_addr("0.0.0.0:50051");
    //创建服务
    IMLoginServicerImpl service;
    //创建ServerBuild
    ServerBuilder builder;
    builder.SetSyncServerOption(ServerBuilder::MIN_POLLERS,3);//设置最小3个
	builder.SetSyncServerOption(ServerBuilder::MAX_POLLERS,4);//设置epoll_wait的个数(即Reactor线程数),设置最大四个，默认最小一个，最大两个
    
    //将地址和服务关联起来
    builder.AddListeningPort(server_addr,grpc::InsecureServerCredentials());
    //保活指令设置(5000ms保活)
    builder.AddChannelArgument(GRPC_ARG_KEEPALIVE_TIME_MS,5000);
    //超时(10000ms)
    builder.AddChannelArgument(GRPC_ARG_KEEPALIVE_TIMEOUT_MS,10000);
    //设置为1即使没有grpc调用也会发送保活包
    builder.AddChannelArgument(GRPC_ARG_KEEPALIVE_PERMIT_WITHOUT_CALLS,1);
    builder.RegisterService(&service);
    
    //启动
    std::unique_ptr<Server> server(builder.BuildAndStart());
    //进入服务循环
    server->Wait();
}

int main(int argc,const char** argv)
{
    RunServer();
    return 0;
}
```

##### client

```c++
#include<iostream>
#include<string>
#include<memory>

#include<grpcpp/grpcpp.h>
#include"IMlogin.grpc.pb.h"

using std::string;
//命名空间Channal
using grpc::Channel;
using grpc::ClientContext;
using grpc::Status;
//自己的命名空间
using IM::Login::IMlogin;
using IM::Login::IMLoginReq;
using IM::Login::IMLoginRes;
using IM::Login::IMRegistReq;
using IM::Login::IMRegistRes;
class ImLoginClient{
public:
    ImLoginClient(std::shared_ptr<Channel> channel)
        :stub_(IMlogin::NewStub(channel)){//为stub赋值

    }
    void Register(const string &username ,const string &password){
        //封装RegistReq
        IMRegistReq request;
        request.set_user_name(username);
        request.set_password(password);
        
        IMRegistRes response;
        ClientContext context;
        std::cout<<"Regist   "<<std::endl;
        Status status = stub_->Regist(&context,request,&response);
        if(status.ok()){
            std::cout<<"user_name "<<response.user_name()<<std::endl<<
            "user_id "<<response.user_id()<<std::endl;
        }else{
            std::cout<<response.result_code()<<std::endl;
        }
    }
    void Login(const string &username ,const string &password){
        //封装RegistReq
        IMLoginReq request;
        request.set_user_name(username);
        request.set_password(password);
        
        IMLoginRes response;
        ClientContext context;
        std::cout<<"Login   "<<std::endl;
        Status status = stub_->Login(&context,request,&response);
        if(status.ok()){
            std::cout<<"user_id "<<response.user_id()<<std::endl;
        }else{
            std::cout<<response.result_code()<<std::endl;
        }
    }
private:
    //Stub
    std::unique_ptr<IMlogin::Stub> stub_;
};

int main()
{
    std::string server_addr="localhost:50051";
    //每个服务器的地址对应一个Channel
    ImLoginClient client(
        grpc::CreateChannel(server_addr,grpc::InsecureChannelCredentials())//创建Channel认证
    );
    string user_name="my";
    string password="123456";
    client.Register(user_name,password);
    client.Login(user_name,password);
    return 0;
}
```

### 异步

```cpp
/*
 *
 * Copyright 2015 gRPC authors.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *     http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 *
 */

#include <iostream>
#include <memory>
#include <string>
#include <thread>

#include "IM.Login.grpc.pb.h"
#include "IM.Login.pb.h"

#include <grpc/support/log.h>
#include <grpcpp/grpcpp.h>

using grpc::Server;
using grpc::ServerAsyncResponseWriter;
using grpc::ServerBuilder;
using grpc::ServerCompletionQueue;
using grpc::ServerContext;
using grpc::Status;

// 自己proto文件的命名空间
using IM::Login::ImLogin;
using IM::Login::IMLoginReq;
using IM::Login::IMLoginRes;
using IM::Login::IMRegistReq;
using IM::Login::IMRegistRes;

class ServerImpl final {
 public:
    ~ServerImpl() {
        server_->Shutdown();
        // Always shutdown the completion queue after the server.
        cq_->Shutdown();
    }

    // There is no shutdown handling in this code.
    void Run() {  // 启动
        std::string server_address("0.0.0.0:50051");

        ServerBuilder builder;
        // Listen on the given address without any authentication mechanism.
        builder.AddListeningPort(server_address, grpc::InsecureServerCredentials());
        // Register "service_" as the instance through which we'll communicate with
        // clients. In this case it corresponds to an *asynchronous* service.
        builder.RegisterService(&service_);
        // Get hold of the completion queue used for the asynchronous communication
        // with the gRPC runtime.
        cq_ = builder.AddCompletionQueue();
        // Finally assemble the server.
        server_ = builder.BuildAndStart();
        std::cout << "Server listening on " << server_address << std::endl;

        // Proceed to the server's main loop.
        HandleRpcs();
    }

 private:
    // Class encompasing the state and logic needed to serve a request.
    class CallData {
    public:
        // Take in the "service" instance (in this case representing an asynchronous
        // server) and the completion queue "cq" used for asynchronous communication
        // with the gRPC runtime.
        CallData(ImLogin::AsyncService* service, ServerCompletionQueue* cq)
            : service_(service), cq_(cq), status_(CREATE) {
            std::cout << "CallData constructing, this: " << this
                    << std::endl;  // darren 增加
            // Invoke the serving logic right away.
            Proceed();
        }
        virtual ~CallData(){}
        virtual void Proceed() {
            // std::cout << "CallData Prceed" << std::endl//;
            return;
        }
        // 通用的
        // The means of communication with the gRPC runtime for an asynchronous
        // server.
        ImLogin::AsyncService* service_;
        // The producer-consumer queue where for asynchronous server notifications.
        ServerCompletionQueue* cq_;
        // Context for the rpc, allowing to tweak aspects of it such as the use
        // of compression, authentication, as well as to send metadata back to the
        // client.
        ServerContext ctx_;
        // 有差异的
        // What we get from the client.
        // IMRegistReq request_;
        // // What we send back to the client.
        // IMRegistRes reply_;

        // // The means to get back to the client.
        // ServerAsyncResponseWriter<IMRegistRes> responder_;

        // Let's implement a tiny state machine with the following states.
        enum CallStatus { CREATE, PROCESS, FINISH };
        CallStatus status_;  // The current serving state.
    };

    class RegistCallData : public CallData {
    public:
        RegistCallData(ImLogin::AsyncService* service, ServerCompletionQueue* cq) 
            :CallData(service, cq), responder_(&ctx_) {
            Proceed();
        }
        ~RegistCallData() {}
        void Proceed() override {
            // std::cout << "RegistCallData Prceed" << std::endl//;
            std::cout << "this: " << this
                    << " RegistCallData Proceed(), status: " << status_
                    << std::endl;   // darren 增加
            if (status_ == CREATE) {  // 0
                std::cout << "this: " << this << " RegistCallData Proceed(), status: "
                            << "CREATE" << std::endl;
                // Make this instance progress to the PROCESS state.
                status_ = PROCESS;

                // As part of the initial CREATE state, we *request* that the system
                // start processing SayHello requests. In this request, "this" acts are
                // the tag uniquely identifying the request (so that different CallData
                // instances can serve different requests concurrently), in this case
                // the memory address of this CallData instance.

                service_->RequestRegist(&ctx_, &request_, &responder_, cq_, cq_, this);//相当于把this和Regist函数绑定并加入cq_,并将request_赋值

            } else if (status_ == PROCESS) {  // 1
                std::cout << "this: " << this << " RegistCallData Proceed(), status: "
                            << "PROCESS" << std::endl;
                // Spawn a new CallData instance to serve new clients while we process
                // the one for this CallData. The instance will deallocate itself as
                // part of its FINISH state.
                new RegistCallData(service_, cq_);  // 1. 创建处理逻辑

                reply_.set_user_name(request_.user_name());
                reply_.set_user_id(10);
                reply_.set_result_code(0);

                // And we are done! Let the gRPC runtime know we've finished, using the
                // memory address of this instance as the uniquely identifying tag for
                // the event.
                status_ = FINISH;
                responder_.Finish(reply_, Status::OK, this);//返回reply_
            } else {
                std::cout << "this: " << this << " RegistCallData Proceed(), status: "
                            << "FINISH" << std::endl;
                GPR_ASSERT(status_ == FINISH);
                // Once in the FINISH state, deallocate ourselves (RegistCallData).
                delete this;
            }
        }
    private:
        IMRegistReq request_;
        IMRegistRes reply_;
        ServerAsyncResponseWriter<IMRegistRes> responder_;
    };

    class LoginCallData : public CallData {
    public:
        LoginCallData(ImLogin::AsyncService* service, ServerCompletionQueue* cq) 
            :CallData(service, cq), responder_(&ctx_) {
            Proceed();
        }
        ~LoginCallData() {}
        void Proceed() override {
            // std::cout << "LoginCallData Prceed" << std::endl//;
            std::cout << "this: " << this
                    << " LoginCallData Proceed(), status: " << status_
                    << std::endl;   // darren 增加
            if (status_ == CREATE) {  // 0
                std::cout << "this: " << this << " LoginCallData Proceed(), status: "
                            << "CREATE" << std::endl;
                // Make this instance progress to the PROCESS state.
                status_ = PROCESS;

                // As part of the initial CREATE state, we *request* that the system
                // start processing SayHello requests. In this request, "this" acts are
                // the tag uniquely identifying the request (so that different CallData
                // instances can serve different requests concurrently), in this case
                // the memory address of this CallData instance.

                service_->RequestLogin(&ctx_, &request_, &responder_, cq_, cq_, this);

            } else if (status_ == PROCESS) {  // 1
                std::cout << "this: " << this << " LoginCallData Proceed(), status: "
                            << "PROCESS" << std::endl;
                // Spawn a new CallData instance to serve new clients while we process
                // the one for this CallData. The instance will deallocate itself as
                // part of its FINISH state.
                new LoginCallData(service_, cq_);  // 1. 创建处理逻辑

                reply_.set_user_id(10);
                reply_.set_result_code(0);

                // And we are done! Let the gRPC runtime know we've finished, using the
                // memory address of this instance as the uniquely identifying tag for
                // the event.
                status_ = FINISH;
                responder_.Finish(reply_, Status::OK, this);
            } else {
                std::cout << "this: " << this << " LoginCallData Proceed(), status: "
                            << "FINISH" << std::endl;
                GPR_ASSERT(status_ == FINISH);
                // Once in the FINISH state, deallocate ourselves (LoginCallData).
                delete this;
            }
        }
    private:
        IMLoginReq request_;
        IMLoginRes reply_;
        ServerAsyncResponseWriter<IMLoginRes> responder_;
    };
    // This can be run in multiple threads if needed.
    void HandleRpcs() {  // 可以运行在多线程
        // Spawn a new CallData instance to serve new clients.
        new RegistCallData(&service_, cq_.get());  //
        new LoginCallData(&service_, cq_.get());
        void* tag;                              // uniquely identifies a request.
        bool ok;
        while (true) {
            // Block waiting to read the next event from the completion queue. The
            // event is uniquely identified by its tag, which in this case is the
            // memory address of a CallData instance.
            // The return value of Next should always be checked. This return value
            // tells us whether there is any kind of event or cq_ is shutting down.
            std::cout << "before cq_->Next "
                    << std::endl;  // 1. 等待消息事件  darren 增加
            GPR_ASSERT(cq_->Next(&tag, &ok));//Next最终实现了epoll_wait，客户端调用哪个函数返回对应绑定的对象
            std::cout << "after cq_->Next " << std::endl;  // darren 增加
            GPR_ASSERT(ok);
            std::cout << "before static_cast" << std::endl;  // darren 增加
            static_cast<CallData*>(tag)->Proceed();
            std::cout << "after static_cast" << std::endl;  // darren 增加
        }
    }

    std::unique_ptr<ServerCompletionQueue> cq_;
    ImLogin::AsyncService service_;
    std::unique_ptr<Server> server_;
};

int main(int argc, char** argv) {
  ServerImpl server;
  server.Run();

  return 0;
}

```

