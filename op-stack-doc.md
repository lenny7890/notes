# Op stack 说明文档

* 本说明基于>= ubuntu20.04 LTS系统

## 目录
- [1. 基础环境准备](###基础环境准备)
- [2. 本地网络搭建](###本地网络搭建)
- [3. Goerli测试网搭建](###Goerli测试网搭建OpStack)
- [4. Op代码说明](##Op代码说明)


### 基础环境准备
* 基于root 账户, 如果是国内服务器，拉取go mod ,docker build ,docker pull 可能存在网络问题，根据情况或设置goproxy 和 [docker proxy](https://docs.docker.com/network/proxy/)即可
* [最新环境说明](https://stack.optimism.io/docs/build/getting-started/#prerequisites)

| software | version | install command |
| ------- | ------- | ------- |
| git, curl, jq, make | os default | `sudo apt install -y git curl make jq` |
| golang | 1.20 | `sudo apt update`<br>`wget https://go.dev/dl/go1.20.linux-amd64.tar.gz`<br>`tar xvzf go1.20.linux-amd64.tar.gz`<br>`sudo cp go/bin/go /usr/bin/go`<br>`sudo mv go /usr/local`<br>`echo export GOROOT=/usr/local/go >> ~/.bashrc` |
| node | 16.x.x | `可以用nvm` <br> `curl -fsSL https://deb.nodesource.com/setup_16.x \| sudo -E bash -` <br> `sudo apt-get install -y nodejs npm` |
| docker | lasted | [docker install](https://docs.docker.com/engine/install/ubuntu/#installation-methods) <br> [docker uninstall old version](https://docs.docker.com/engine/install/ubuntu/#uninstall-old-versions) |
| pnpm | 16.x.x | `sudo npm install -g pnpm` |
| foundry | lasted | [推荐官方安装方法](https://book.getfoundry.sh/getting-started/installation) |


1. 更新 apt 环境
```shell
$ apt update
$ apt upgrade
```
2. 安装所需系统组件
```
$ apt install -y gcc g++ git curl make jq
```
golang(不按照这个也可以，只要保证golang 安装为1.20版本, go install 可用就可以)
```
$ cd ~ && wget https://go.dev/dl/go1.20.linux-amd64.tar.gz && tar xvzf go1.20.linux-amd64.tar.gz
$ cp go/bin/go /usr/bin/go && mv go /usr/local
$ echo export GOROOT=/usr/local/go >> ~/.bashrc
$ echo export GO_PATH=~/go >> ~/.bashrc
$ echo export PATH="$PATH:$GO_PATH/bin" >> ~/.bashrc
$ go version
```
node
```
$ curl -fsSL https://deb.nodesource.com/setup_16.x \| sudo -E bash -
$ apt-get install -y nodejs npm
$ npm install -g pnpm
```
foundry
```
$ curl -L https://foundry.paradigm.xyz | bash
$ source .bashrc
$ foundryup
```

3. 获取op 代码
```
$ cd ～ && git clone https://github.com/ethereum-optimism/optimism.git
$ cd optimism
$ pnpm i
```
4. build 代码
```
$ cd ~/optimism
$ make op-node op-batcher op-proposer
$ pnpm build
```


### 本地网络搭建
* [详细说明](https://github.com/ethereum-optimism/optimism/blob/65ec61dde94ffa93342728d324fecf474d228e1f/specs/meta/devnet.md)
```
$ cd ~/optimism
# starts the devnet
$ make devnet-up
# stops the devnet
$ make devnet-down
# removes the devnet by deleting images and persistent volumes
$ make devnet-clean 
```
#### 其他说明

- 要开始，请运行（在monorepo的根目录中）`make devnet up`. 它第一次运行时会相对较慢，因为它需要下载docker image，之后会更快

- 要停止，请运行（在 monorepo 的根目录中）`make devnet-down`.

- 要清理所有内容，请运行（在 monorepo 的根目录中）`make devnet-clean`.

- [L1 合约地址](https://github.com/ethereum-optimism/optimism/blob/65ec61dde94ffa93342728d324fecf474d228e1f/packages/contracts-bedrock/deploy-config/devnetL1.json). L2 合约地址是标准地址.

- 开发网络和正式网络区别:

| Parameter | Real-world | Devnode|
| ------- | ------- | ------- |
| L1 chain ID |	1 |	900 |
| L2 chain ID |	10 | 901 |
| Time between L1 blocks (in seconds) |	12 | 3 |

Test ETH:
```
Address: 0xf39fd6e51aad88f6f4ce6ab8827279cfffb92266
Private key: ac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80
```


### Goerli测试网搭建OpStack
* [详细说明](https://stack.optimism.io/docs/build/getting-started/#get-access-to-a-goerli-node)

```

```


### 4.Op代码说明
内容章节三的内容在这里。
