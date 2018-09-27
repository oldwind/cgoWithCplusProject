### cgo-cpp-php

#### 0x01 背景

语言和语言之间的通讯，本身是一个比较复杂的问题，要考虑到数据格式、通讯协议等种种问题，此次，我在项目中需要实现一个php和go之间的通讯，总结起来，通讯方式有三种
1. 采用http协议的方式，这中方式比较常见，go启动一个http服务，php通过curl方式去调用
2. 采用gcpc方式，gRPC是一个高性能、通用的开源RPC框架，其由Google主要面向移动应用开发并基于HTTP/2协议标准而设计，基于ProtoBuf(Protocol Buffers)序列化协议开发，且支持众多开发语言。但是hhvm和php都需要插件支持，php 5.X暂时应该没有官方插件
3. php的扩展调用cgo写的动态连接库，这种方式用的比较少，但是，此次项目中却必须采用这种方式。有多个原因
	
	- php端上要采用复杂的加密方式，go的加密库中有php插件中暂时不存在的。
	- grpc的原因，Go的服务端目前提供的Grpc服务，php由于版本问题，暂时无法使用

综上，选择第三种方法是迫不得已，况且开发周期非常短，也就三天左右	

#### 0x02 关于Cgo

我们第一个情况是要将go打成可以被C++调用的动态链接库, 那么必然选择cgo，在go中import C，需要关注几点
	
	 package main

	 import "C" 
	 
	 import (
		 "unsafe"
		 "fmt"
	 )
	 
	 var ret [10240]byte
 
	// export和注释符号//不能存在空格
	//export myFunc
	func myFunc(param1 string) (*C.char, int32) {
		str := []byte("test return")
		copy(ret[:], str);
		retP := (*C.char)(unsafe.Pointer(&ret))

		defer println(retP)
		return retP, int32(4)
	}
 
	func main() {}

