# Caddy Manager

一个 Caddy 管理应用。

- manager 管理面板（含自身节点）
- node 单节点
# 安装指南

确保需要部署的面板鸡和落地鸡都已经安装Docker Compose,全新环境可使用 ```curl -sSL https://get.docker.com/ | sh``` 一键安装

## manger

- 1-下载整个manager文件夹到你需要部署的面板鸡上
- 2-获取 Github/Jihulab 的 Client ID 和密钥
- 3-修改 ```.env.example```中的各项配置，并重命名 ```.env.example```为```.env```
- 4-确保 80、443、8080 端口可用 
#### manger目录检查
你的目录应该包括如下所有文件：
```
manager/
├── caddy
│   └── Caddyfile
├── docker-compose.yml
└── manager

```

#### 获取Client ID、Secret

Github、Gitlab、Jihulab、Gitee 作为后台管理员账号  
- 首先我们需要新建一个验证应用，以 Github 为例，登录 Github 后，打开 https://github.com/settings/developers ，依次选择“OAuth Apps” - “New OAuth App”    
`Application name` - 随意填写  
`Homepage URL` - 填写面板的访问域名，如："http://example.com"  
`Authorization callback URL` - 填写回调地址，如："http://cdn.example.com/oauth2-callback"  
- 点击 “Register application” 
- 保存页面中的 Client ID，然后点击 “Generate a new client secret“，创建一个新的 Client Secret，新建的密钥仅会显示一次，请妥善保存
<br/>
- JihuLab 的应用创建入口为：https://jihulab.com/-/profile/applications  
- `Redirect URL` 中应填入回调地址  
- 在下方`范围`中勾选 `read_user` 和 `read_api` 
- 创建完成后，保存好应用程序 ID 和密码

#### 修改配置
- 将上一步获取的 id、secret 替换env中的 client_id、client_secret
- 设置或随机生成一个密码（https://suijimimashengcheng.bmcx.com/） 替换env中的 node_secret
- 打开https://api.github.com/users/{你的github用户名} 获取你的id 替换env中的000000
- 修改Caddyfile，如下 （如果面板鸡不当节点请删除 ==== 分割的内容）
```
{
    #debug
    order forward_proxy before reverse_proxy
}
# ==================不当节点删除我================
# 可选节点，:443 是必须的，不能删除
:443, node.caddy-manager.com {
    tls webmaster@caddy-manager.com
    forward_proxy {
        hide_ip
        hide_via
        probe_resistance www.baidu.com
        dashboard {
            use_tls false
            grpc manager.caddy-manager.com:8080 # 当节点记得修改这里
            server_id 6dc58acd-3712-4231-b7c2-1ce042b0a72e # 获取到的节点UUID
            secret_key 969a57c2-b63d-4c31-97a0-e908e66576cb # .env中写入的 node_secret
        }
    }
    # 伪装站点
    reverse_proxy https://google.github.io {
        header_up Host {upstream_hostport}
    }
}
# ==================不当节点删除我================

# 管理面板站点 manager.caddy-manager.com 请改成你自己的域名并且添加对应解析 ，webmaster@caddy-manager.com 可选择修改
manager.caddy-manager.com {
    tls webmaster@caddy-manager.com
    reverse_proxy manager:8080
}

``` 
#### 运行面板
- 进入manager目录后运行 ```docker-compose up -d``` 打开  你修改 manager.caddy-manager.com 后的域名 登录即可进入面板
- ![DEMO](images/manager.png)

## node

- 1-下载整个node文件夹到你需要部署的鸡上
- 2-获取 manager生成的 UUID
- 3-修改 ```Caddyfile```中的各项配置
- 4-确保 80、443 端口可用 

#### node目录检查
你的目录应该包括如下所有文件：
```
node/
├── caddy
│   └── Caddyfile
└── docker-compose.yaml

```
#### 生成UUID
- 登录上一步中搭建好的面板，进入Nodes,点击ADD，填入相关信息
- ![INFO](images/info.png)
- 复制生成的UUID准备修改Caddyfile
- ![UUID](images/uuid.png)
- 参照注释修改 Caddyfile
```
{
    order forward_proxy before reverse_proxy
}

# 节点，:443 是必须的，不能删除   node2.caddy-manager.com 修改为你自己的域名 webmaster@caddy-manager.com 选择修改
:443, node2.caddy-manager.com {
    tls webmaster@caddy-manager.com
    forward_proxy {
        hide_ip
        hide_via
        probe_resistance nb.com
        dashboard {
            use_tls false
            grpc manager.caddy-manager.com:8080 # 上一步搭建面板鸡的信息
            server_id 6dc58acd-3712-4231-b7c2-1ce042b0a72e # 获取到的节点UUID
            secret_key 969a57c2-b63d-4c31-97a0-e908e66576cb # .env中写入的 node_secret
        }
    }
    # 伪装站点
    reverse_proxy https://google.github.io {
        header_up Host {upstream_hostport}
    }
}

# 还可以托管其他站点

```
#### 运行节点
- 进入node目录后运行 ```docker-compose up -d``` 
- 相关客户端配置请谷歌

## 用户配置
- 登录上一步中搭建好的面板，新增Plans，请填写两遍，确保plan下有node
- ![plan](images/plan.png)
- ![plan1](images/plan1.png)
- 进入Users给用户分配Plan UUID、设置流量限制，即可通畅上网 
- ![user](images/user.png)
