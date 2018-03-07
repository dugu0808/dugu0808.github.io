---
layout:     post
title:      "深入学习Fabric链码Chaincode"
subtitle:   "区块链学习笔记"
date:       2018-03-07
author:     "Liu Ke"
header-img: "img/BC.jpg"
tags:
    - Blockchain区块链
    - Hyperledger超级账本
---

> 参考资料：《区块链 原理、设计与应用》 杨保华, 陈昌编著

链码（chaincode）或者链上代码，是Fabric中十分关键的一个概念。链码源自智能合约的思想，并进行了进一步扩展，支持多种高级编程语言。

目前Fabric项目中提供了用户链码和系统链码。前者运行在单独的容器中，提供对上层应用的支持，后者则嵌入在系统内，提供对系统进行配置、管理的支持。

一般所谈的链码为用户链码，通过提供可编程能力提供了对上层应用的支持。用户通过链码相关的API编写用户链码，即可对账本中的状态进行更新操作。

链码会对Fabric应用程序发送的交易做出响应，执行代码逻辑，与账本进行交互。区块链网络中的成员商定业务逻辑后，可将业务逻辑编程到链码中，然后大家遵循合约来执行。

链码会创建一些状态（state）并写入账本中。状态带有绑定到链码的命名空间，并且仅限于创建它的的链码所使用，不能被其他链码直接访问。但是，在合适的许可范围内，一个链码也可以调用另一个链码，间接访问其状态。另外，在一些场景下，不仅需要访问状态的当前值，还需要能够查询状态所有历史值，这就存放账本状态的数据库提出了更多的要求。

链码最核心的机构为`ChaincodeSpec`，对链码的部署和调用都基于该结构进行进一步封装（`ChaincodeDeploymentSpec`和`ChaincodeInvocationSpec`）,链码信息至少需要指定名称、版本号和实例化策略。


![](https://raw.githubusercontent.com/dugu0808/dugu0808.github.io/master/img/in-post/180307/%E9%93%BE%E7%A0%81%E7%9B%B8%E5%85%B3%E7%BB%93%E6%9E%84.png)

链码经过安装和实例化操作之后，即可被调用。在安装的时候，需要指定安装到哪个Peer节点(Endorser);实例化的时候还需要指定是在哪个通道内进行实例化。链码之间还可以通过互相调用，创建更为灵活的应用逻辑。

链码在Fabric节点上的隔离沙盒（目前为Docker容器）中执行，并通过gRPC协议来与节点进行交互。必要的交互包括调用链码、读写账本、返回响应结果等。Fabric目前主要支持Go语言的链码。

## gRPC消息协议

Fabric中大量采用了gRPC消息在不同组件之间进行通信交互，主要包括如下几种情况：

- 客户端访问Peer节点

- 客户端和Peer访问Orderer节点

- 链码容器跟Peer节点之间通信

- 多个Peer节点之间的通信

对于链码容器和Peer节点之间的操作，链码容器启动后，会向Peer节点进行注册，gRPC地址为`/protos.ChaincodeSupport/Register`。消息为ChaincodeMessage结构，如下图所示。（定义在`protos/peer/chaincode_shim.proto`文件）。其中，Payload域中可以包括各种Chaincode操作消息，如`GetHistoryForKey`、`GetQueryResult`、`PutStateInfo`、`GetStateByRange`等。
注册完成后，双方建立起双工通道，通过更多消息类型来实现多种交互。

![](https://raw.githubusercontent.com/dugu0808/dugu0808.github.io/master/img/in-post/180307/ChaincodeMessage%E6%B6%88%E6%81%AF%E7%BB%93%E6%9E%84.png)

## 用户链码

用户链码相关的代码都在`core/chaincode`路径下。其中`core/chaincode/shim`包中的代码主要是供链码容器侧调用使用，其他代码主要是Peer侧使用。

### Chaincode接口
Fabric中为链码提供了很好的封装支持，编写链码还是相对比较简单。以Golang为例，每个链码都需要实现以下Chaincode接口：

```
type Chaincode interface{

Init(stub ChaincodeStubInterface) pb.Response

Invoke(stub ChaincodeStubInterface) pb.Response

}

```

其中:

- `Init`:当链码收到实例化（instantiate）或升级（upgrade）类型的交易时，Init方法会被调用。

- `Invoke`:当链码收到调用（invoke）或查询（query）类型的交易时，Invoke方法会被调用。

###链码结构
一个链码的必要结构如下所示，在其中利用`shim.ChaincodeStubInterface`结构，实现跟账本的交互逻辑：

```
    package main
    
    //引入必要的包
    
    import(
    
    	"github.com/hyperledger/fabric/core/chaincode/shim"
    
    	pb "github.com/hyperledger/fabric/protos/peer"
    
    )
    
    //声明一个结构体
    
    type SimpleChaincode struct {}
    
    
    //为结构体添加Init方法
    
    func (t *SimpleChaincode) Init(stub shim.ChaincodeStubInterface) pb.Response{
    
    	//在该方法中实现链码运行中初始化或升级的处理逻辑
    
    	//编写时可灵活使用stub中的API
    
    }
    
    //为结构体添加Invoke方法
    
    func (t *SimpleChaincode) Invoke(stub shim.ChaincodeStubInterface) pb.Response{
    
    	//在该方法中实现链码运行中被调用或查询时的处理逻辑
    
    	//编写时可灵活使用stub中的API
    
    }
    
    //主函数，需要调用shim.Start()方法
    
    func main() {
    
    	err := shim.Start(new(SimpleChaincode))
    
    	if err != nil {
    
    		fmt.Printf("Error start Simple chaincode : %s", err)
    
    	}
    
    }

```