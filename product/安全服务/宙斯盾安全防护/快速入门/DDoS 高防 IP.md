 DDoS 高防 IP 产品通过代理转发的模式，为互联网业务服务端提供大流量 DDoS 攻击的防护和流量清洗服务。用户可以通过简单配置，将业务流量和攻击流量指向 DDoS 高防 IP，攻击流量在经过高防 IP 时，会由防护系统进行检测和清洗，并将清洁业务流量转发回源到业务服务端，以保障业务在 DDoS 攻击场景下的可用性。
 
本文将详细说明 DDoS 高防 IP 配置上线的步骤。关于如何选择购买配置，详情请参见 [**产品配置说明**](https://cloud.tencent.com/document/product/685/18798) 。

### 流程图
![](https://main.qcloudimg.com/raw/7742fd0997412f9957a83cc62dffb6e7.png)

### 接入步骤
1. **购买 DDoS 高防 IP**
a. 用户进入 [宙斯盾高防控制台](https://console.cloud.tencent.com/gamesec)，在左侧目录中，单击【DDoS 高防 IP】，在 “高防 IP 列表” 下，单击 【购买】。
![](https://main.qcloudimg.com/raw/3438b0feb57b8a69c9579a7eef5b3843.png)
b. 根据业务需求选取需要的配置，确认好配置，单击【立即支付】。
![](https://i.imgur.com/11lF1ZC.png)

2. **添加高防 IP 转发规则组**
a. 在 “DDoS 高防 IP” 下，单击【转发规则组】，单击【添加转发规则组】。
b. 输入 “新规则组名称”，选择好 “转发模式” 和 “所属项目” ，单击【确定】。
![](https://main.qcloudimg.com/raw/a7d6fed5e864c5922534b4433c138fe3.png)
>**说明：**
>每个转发规则组下默认免费提供 60 个转发规则。

3. **添加转发规则**
a. 创建转发规则组后，单击【转发规则组 ID】，进入 “转发规则组详情” 页面。
![](https://main.qcloudimg.com/raw/6fce19de8bcd05382f20c5f4fb98516f.png)
b. 在【规则组信息】下，单击【添加转发规则】，可以添加基于 4 层端口转发的规则。请根据业务协议需求，选择 “协议类型”，配置 “会话保持” 开关，选取适合的 “轮询策略”，并输入 “转发端口” 号、“源站端口” 号和 “源站公网 IP 和权重”，创建转发规则支持批量添加，单击【确定】。
![](https://main.qcloudimg.com/raw/7b40fa357f1bf439616b324d66ea4846.png)
>**说明：**
每个转发规则可以配置 40 个源站公网 IP。

 **协议类型**：选取服务的端口的协议类型，支持 TCP 或 UDP 协议。
 **会话保持**：开启会话保持是保证一系列相关连的访问请求会分配到同一台服务器上。当同一个转发规则下，有多个源站 IP 时，若需要同一个客户的请求由同一个服务器处理，则需要开启会话保持。
 **轮询策略**：当同一个转发规则下，有多个源站 IP 时，可以选取按权重轮询或按最小连接数轮询来分配后端源站服务器处理业务请求的比重。权重轮询，是按照源站 IP 权重大小进行分配；最小连接数轮询，按照公网 IP 最小连接数进行分配。
  **转发端口**：指需要在高防 IP 上提供服务的端口号。客户端或用户的业务请求会直接访问高防 IP 的该端口。
  **源站端口**：指在源站服务器提供服务的端口号，与转发端口对应。高防 IP 会将用户在转发端口的业务请求转发到源站服务器 IP 的该源站端口处理。
 **源站公网 IP 和权重**：指提供服务的后端源站服务器的 IP 地址。当有多个源站公网 IP 时，高防 IP 会将业务请求轮询到每个源站 IP，设置的权重的大小代表着该源站 IP 处理的业务请求比例，权重越大则收到的业务请求数越多。
>**注意：**
> - 如果有多条转发规则，可以批量创建。分别在转发端口和源站端口输入框输入多个端口号，用逗号分隔。转发规则将按端口顺序关系一一创建。
> - 如下端口号为转发集群保留端口，不可添加为转发端口：843，3306，1433，1434，36000，56000，3389。

4. **为高防 IP 绑定转发规则组**
创建完转发规则组后，在 “DDoS 高防 IP” 下，单击【转发规则组】，在 “操作” 列下，单击【关联高防 IP】，将该转发规则组绑定到高防 IP。
  ![](
https://main.qcloudimg.com/raw/a967def3d82e72387fe4179d101e7228.png)
也可以在 “DDoS 高防 IP” 下选择 “高防 IP”，进入 “DDoS 高防 IP 详情” 页面， 单击【基础配置】，在 “转发规则组设置” 项目下，单击【绑定】，为高防 IP 绑定转发规则组。
 ![](https://main.qcloudimg.com/raw/8d6614440b2c3cedd8f899322713af29.png)
 ![](https://main.qcloudimg.com/raw/bfc70be9358c4c622c6e5ad49e235d06.png)

5. **业务指向高防 IP**
 为高防 IP 绑定转发规则组后，用户可以验证从高防 IP 的转发端口到源站服务器的源站端口的连通性。配置业务指向到高防 IP，即完成了高防 IP 的接入配置。如用户业务之前使用了域名解析，用户可在业务域名解析服务商中更改域名解析，将原 IP 地址更换为绑定的高防 IP 地址。
 以腾讯云域名解析为例：
![](https://main.qcloudimg.com/raw/351f9c5355dd4e41e5e5ef0cd3537a7a.png)
完成上面的配置后，用户的源站服务器就已经处于高防 IP 的防护之下了。

6. **CNAME 防护域名**
用户还可以创建免费的防护域名，通过在业务域名解析服务商添加 CNAME 解析到防护域名接入。配置详情请参见 [**防护域名绑定高防IP**](https://cloud.tencent.com/document/product/685/18808)。






