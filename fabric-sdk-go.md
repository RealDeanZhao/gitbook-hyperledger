---
description: fabric-sdk-go常用包的用法讲解
---

# fabric-sdk-go

## pkg/fabsdk包

```go
import "github.com/hyperledger/fabric-sdk-go/pkg/fabsdk"
```

### fabsdk.FabricSDK

### fabsdk.New

**返回:** fabsdk.FabricSDK实例

**参数:** [core.ConfigProvide](fabric-sdk-go.md#core-configprovider)

{% hint style="info" %}
fabsdk.FabricSDK对每个组织实例化一次即可
{% endhint %}

## pkg/client/resmgmt包

```go
import "github.com/hyperledger/fabric-sdk-go/pkg/client/resmgmt"
```

### resmgmt.Client

用来管理诸如chaincode, channel等资源, 需要管理员身份.

#### 针对chaincode的管理:

* install
* upgrade
* instantiate

#### 针对channel的管理:

* join
* save \(create以及update\)

{% hint style="info" %}
在invoke chaincode的时候, 实际上使用的是[channel.Client](fabric-sdk-go.md#channel-client)
{% endhint %}

### resmgmt.New

**返回:** resmgmt.Client实例

**参数:** context.ClientProvider

context.ClientProvider需要通过[fabsdk.FabricSDK](fabric-sdk-go.md#fabsdk-fabricsdk)和[msp.Client](fabric-sdk-go.md#msp-client)生成.

{% hint style="info" %}
resmgmt.Client对每个组织实例化一次即可

需要管理员身份
{% endhint %}

**使用方法如下:**

```go
// import "github.com/hyperledger/fabric-sdk-go/pkg/client/resmgmt"

// mspClient为msp.Client实例
adminIdentity, err := mspClient.GetSigningIdentity("admin")
// fabricsdk为fabsdk.FabricSDK实例
adminContext := fabricsdk.Context(fabsdk.WithIdentity(adminIdentity))

resmgmtClient, err := resmgmt.New(adminContext)
```

## pkg/client/channel包

```go
import "github.com/hyperledger/fabric-sdk-go/pkg/client/channel"
```

### channel.Client

用于调用\(execute, query\)某个channel的chaincode

### channel.New

**返回:** channel.Client实例

**参数:** context.ChannelProvider作为参数

**使用方法如下:**

```go
// import "github.com/hyperledger/fabric-sdk-go/pkg/client/channel"
// "github.com/hyperledger/fabric-sdk-go/pkg/fabsdk"

// fabricsdk为fabsdk.FabricSDK实例
channelContext := fabricsdk.ChannelContext(channel, fabsdk.WithUser(username), fabsdk.WithOrg(orgname))
channelClient, err := channel.New(channelContext)
```

## pkg/client/msp包

### msp.Client

用来给组织注册以及登记用户

```go
import "github.com/hyperledger/fabric-sdk-go/pkg/client/msp"
```

### msp.New

**返回:** msp.Client实例

**参数:** context.ClientProvider作为参数

**使用方法如下:**

```go
// import "github.com/hyperledger/fabric-sdk-go/pkg/client/msp"

// fabricsdk为fabsdk.FabSDK实例
ctxProvider := fabricsdk.Context()
mspClient, err := msp.New(ctxProvider)
```

## core.ConfigProvider

用于读取配置文件\(network-config.yaml\), [fabsdk.New](fabric-sdk-go.md#fabsdk-new)方法的使用需要传入此参数. 

{% hint style="info" %}
请注意它的类型实际上是一个函数 

```go
type ConfigProvider func() ([]ConfigBackend, error)
```
{% endhint %}

**使用方法如下:**

```go
// import "github.com/hyperledger/fabric-sdk-go/pkg/core/config"
// import "github.com/hyperledger/fabric-sdk-go/pkg/util/pathvar"
cp := config.FromFile(pathvar.Subst("network-config.yaml"))
```

如果想要支持多个环境的配置, 如本地, 开发, 测试, 生产等环境, 我们可以采用override的模式. 

**使用方法如下:**

```go
// import "github.com/hyperledger/fabric-sdk-go/pkg/core/config"
// import "github.com/hyperledger/fabric-sdk-go/pkg/util/pathvar"
// import "github.com/hyperledger/fabric-sdk-go/pkg/common/providers/core"

// local-orderer-peer-ca.yaml示例讲解请参考local-orderer-peer-ca.yaml章节
// 读取本地开发环境的orderer, peer, ca的配置信息
cp := config.FromFile(pathvar.Subst("local-orderer-peer-ca.yaml"))
localBackends, err := cp()

// local-entity-matchers.yaml示例讲解请参考local-entity-matchers.yaml章节
// 读取本地开发环境的entity matcher配置信息
cp := config.FromFile(pathvar.Subst("local-entity-matchers.yaml"))
localMatcherBackends, err := cp()

// 读取network-config.yaml的配置信息
cp := config.FromFile(pathvar.Subst("network-config.yaml"))
networkBackends, err := cp()

// 然后将刚才生成出来的backends组合起来生成一个新的core.ConfigProvider
// 注意有先后顺序, 需要覆盖的部分要放在backends数组中最前面
return func() ([]core.ConfigBackend, error) {
    // 有先后顺序
    localBackends := append(localBackends, mathcerBackends...)
    // 有先后顺序
    localBackends := append(localBackends, networkBackends...)
    return localBackends, nil
}
```

