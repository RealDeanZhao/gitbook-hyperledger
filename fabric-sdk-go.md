# fabric-sdk-go

## fabsdk.FabricSDK

### fabsdk.New

通过fabsdk.New方法来生成实例, New方法里面需要传入[core.ConfigProvider](fabric-sdk-go.md#core-configprovider).

_fabsdk.FabricSDK对每个组织实例化一次即可_

## resmgmt.Client

用来管理诸如chaincode, channel等资源, 需要管理员身份.

#### 针对chaincode的管理:

* install
* upgrade
* instantiate

#### 针对channel的管理:

* join
* save \(create以及update\)

_注意invoke chaincode实际上是用的channel.Client_

### resmgmt.New

通过resmgmt.New方法来生成实例, New方法里面需要传入context.ClientProvider. 

context.ClientProvider需要通过[fabsdk.FabricS](fabric-sdk-go.md#fabsdk-fabricsdk) 和[msp.Clien](fabric-sdk-go.md#msp-client) 生成.

_resmgmt.Client对每个组织实例化一次即可_

使用方法如下:

```text
// mspClient为msp.Client实例
adminIdentity, err := mspClient.GetSigningIdentity("admin")
// fabricsdk为fabsdk.FabricSDK实例
adminContext := fabricsdk.Context(fabsdk.WithIdentity(adminIdentity))

resmgmtclient, err := resmgmt.New(adminContext)
```

## channel.Client

## msp.Client

## core.ConfigProvider

用于读取配置文件\(network-config.yaml\), [fabsdk.New](fabric-sdk-go.md#fabsdk-new)方法的使用需要传入此参数. 

请注意它的类型实际上是一个函数:

```go
type ConfigProvider func() ([]ConfigBackend, error)
```

基本用法如下:

```go
// import "github.com/hyperledger/fabric-sdk-go/pkg/core/config"
// import "github.com/hyperledger/fabric-sdk-go/pkg/util/pathvar"
cp := config.FromFile(pathvar.Subst("network-config.yaml"))
```

如果想要支持多个环境的配置, 如本地, 开发, 测试, 生产等环境, 我们可以采用override的模式. 

如下代码演示了本地环境的配置文件覆盖读取:

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

