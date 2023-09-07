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

3. 获取op 代码并build

optimism
```
$ cd ～ && git clone https://github.com/ethereum-optimism/optimism.git
$ cd optimism
$ pnpm i
$ make op-node op-batcher op-proposer
$ pnpm build
```

op-geth
```
$ cd ~ && git clone https://github.com/ethereum-optimism/op-geth.git
$ cd op-geth
$ make geth
```

### 本地网络搭建
* [详细说明](https://github.com/ethereum-optimism/optimism/blob/65ec61dde94ffa93342728d324fecf474d228e1f/specs/meta/devnet.md)
```
$ cd ~/optimism
$ make install-geth
# starts the devnet
$ make devnet-up
# stops the devnet
$ make devnet-down
# removes the devnet by deleting images and persistent volumes
$ make devnet-clean 
```
#### 其他说明

- 开始，请运行（在monorepo的根目录中）`make devnet up`. 它第一次运行时会相对较慢，因为它需要下载docker image，之后会更快
- 停止，请运行（在 monorepo 的根目录中）`make devnet-down`.
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
  
由于我们正在将OP Stack链部署到Goerli，需要访问Goerli L1节点。可以使用像[Alchemy这样的节点提供程序](https://www.alchemy.com/)，也可以运行[自己的Goerli节点](https://notes.ethereum.org/@launchpad/goerli)。

1. 生成所需地址
在设置链时，需要四个帐户及其私钥：
- Admin: 升级合同的管理员帐户。
- Batcher: 交易数据发布到L1的帐户
- Proposer: 向L1发布L2交易结果的帐户
- Sequencer: 在p2p网络上sign block的帐户。
可以用在contracts-bedrock包中的rekey工具生成,但是不能用于正式链：

```
$ cd ~/optimism/packages/contracts-bedrock

echo "Admin:"
cast wallet new
echo "Proposer:"
cast wallet new
echo "Batcher:"
cast wallet new
echo "Sequencer:"
cast wallet new
```
结果：
```
Admin:
Successfully created new keypair.
Address:     0x9f92bdF0db69264462FC305913960Edfcc7a7c7F
Private key: 0x30e66956e1a12b81f0f2cfb982286b2f566eb73649833831d9f80b12f8fa183c
Proposer:
Successfully created new keypair.
Address:     0x31dE9B6473fc47af36ec23878bA34824B9F4AB30
Private key: 0x8bd1c8dfffef880f8f9ab8162f97ccd119c1aac28fe00dacf919459f88e0f37d
Batcher:
Successfully created new keypair.
Address:     0x6A3DC843843139f17Fcf04C057bb536A421DC9c6
Private key: 0x3ce44144b7fde797a28f4e47b210a4d42c3a3b642e538b54458cba2740db5ac2
Sequencer:
Successfully created new keypair.
Address:     0x98C6cadB1fe77aBB7bD968fC3E9b206111e72848
Private key: 0x3f4241229bb6f155140d98e0f5dd2aad7ae983f5af5d61555d05eb8e5d9514db
```
将这些帐户及其各自的私钥保存一下，后边会用到。用少量的 Goerli ETH 为管理员地址提供资金，因为我们将使用该帐户来部署我们的智能合约。您还需要为Proposer和Batcher程序地址提供资金——请注意，Batcher程序会消耗最多的 ETH，因为它将交易数据发布到 L1。
大约：
Admin — 2 ETH
Proposer — 5 ETH
Batcher — 10 ETH

2. 配置网络参数
```
$ cd ~/optimism/packages/contracts-bedrock
$ cp .envrc.example .envrc
```
修改这些参数
ETH_RPC_URL: L1 节点rpc url
PRIVATE_KEY: 前边生成的Admin地址
DEPLOYMENT_CONTEXT: 网络名称

3. 载入参数

```
$ eval "$(direnv hook bash)"

$ direnv allow .
```
需要选择一个 L1 块作为 Rollup 的起点。最好使用最终确定的 L1 块作为我们的起始块。您可以使用`cast` Foundry 提供的命令来获取所有必要的信息
```
$ cast block finalized --rpc-url $ETH_RPC_URL | grep -E "(timestamp|hash|number)"
```
所示的结果：
```
hash                 0x784d8e7f0e90969e375c7d12dac7a3df6879450d41b4cb04d4f8f209ff0c4cd9
number               8482289
timestamp            1676253324
```

填写在以下位置找到的预填充配置文件的其余部分deploy-config/getting-started.json 。使用配置文件中的默认值并进行以下修改：  

替换"ADMIN"为您之前生成的管理员帐户的地址。  
替换"PROPOSER"为您之前生成的提案者帐户的地址。  
替换"BATCHER"为您之前生成的 Batcher 帐户的地址。  
替换"SEQUENCER"为您之前生成的 Sequencer 帐户的地址。  
替换"BLOCKHASH"为您从命令中获得的块哈希cast。  
替换TIMESTAMP为从 cast 命令中获取的时间戳。这个是数字非字符串  


4. 部署L1合约
配置完网络后，就可以部署链功能所需的 L1 智能合约了。
- 创建`getting-started`部署目录
`mkdir deployments/getting-started`
- 部署 L1 智能合约
```
forge script scripts/Deploy.s.sol:Deploy --private-key $PRIVATE_KEY --broadcast --rpc-url $ETH_RPC_URL
forge script scripts/Deploy.s.sol:Deploy --sig 'sync()' --private-key $PRIVATE_KEY --broadcast --rpc-url $ETH_RPC_URL
```

5. 配置L2文件
已经设置了 L1 端，但现在我们需要设置 L2 端。我们通过生成三个重要文件来做到这一点：一个genesis.json文件、一个rollup.json配置文件和一个jwt.txt JSON Web 令牌 这使得op-node和op-geth能够安全地通信。
- 使用op-node
- 运行以下命令，并确保替换<RPC>为您的 L1 RPC URL
```
cd ~/optimism/op-node

go run cmd/main.go genesis l2 \
    --deploy-config ../packages/contracts-bedrock/deploy-config/getting-started.json \
    --deployment-dir ../packages/contracts-bedrock/deployments/getting-started/ \
    --outfile.l2 genesis.json \
    --outfile.rollup rollup.json \
    --l1-rpc <RPC>

```

- jwt.txt生成文件(如果有没有openssl 安装一下)
```
openssl rand -hex 32 > jwt.txt
```
- 需要将genesis.json文件和jwt.txt文件复制到其中op-geth，以便我们可以使用它来初始化并op-geth在一分钟内运行：
```
cp genesis.json ~/op-geth
cp jwt.txt ~/op-geth
```

6. 初始化op-geth
将运行一个 Sequencer 节点，因此我们需要导入Sequencer之前生成的私钥。我们的序列器将使用该私钥来签署新块

- 创建数据目录文件夹
```
cd ~/op-geth
mkdir datadir
```
- 需要op-geth使用之前生成并复制的创世文件进行初始化
```
build/bin/geth init --datadir=datadir genesis.json
```

7. 运行node相关组件
op-geth和op-node必须在每个节点上运行
op-batcher，op-proposer仅在一个位置运行，即接受事务的定序器。

- 设置这些环境变量

| variable | value |
| ------- | ------- |
| SEQ_KEY | Sequencer账户私钥 |
| BATCHER_KEY | 账户私钥Batcher，至少1 ETH |
| PROPOSER_KEY | Proposer账户私钥 |
| L1_RPC | 正在使用的 L1（例如 Goerli）的 URL |
| RPC_KIND | 您连接的 L1 服务器的类型，可以优化请求。可用选项有alchemy, quicknode, parity, nethermind, debug_geth, erigon, basic, 和any |
| L2OO_ADDR | L2OutputOracleProxy的地址，可在~/optimism/packages/contracts-bedrock/deployments/getting-started/L2OutputOracleProxy.json |


- op-geth
```
cd ~/op-geth

./build/bin/geth \
        --datadir ./datadir \
        --http \
        --http.corsdomain="*" \
        --http.vhosts="*" \
        --http.addr=0.0.0.0 \
        --http.api=web3,debug,eth,txpool,net,engine \
        --ws \
        --ws.addr=0.0.0.0 \
        --ws.port=8546 \
        --ws.origins="*" \
        --ws.api=debug,eth,txpool,net,engine \
        --syncmode=full \
        --gcmode=archive \
        --nodiscover \
        --maxpeers=0 \
        --networkid=42069 \
        --authrpc.vhosts="*" \
        --authrpc.addr=0.0.0.0 \
        --authrpc.port=8551 \
        --authrpc.jwtsecret=./jwt.txt \
        --rollup.disabletxpoolgossip=true
```

有些情况是需要重置op-geth，如下:

+ 重新初始化 op-geth
有几种情况表明数据库损坏并要求您重置op-geth组件：

 - op-node首次启动和退出时出错时。

当op-node发出此错误时：
```
stage 0 failed resetting: temp: failed to find the L2 Heads to start from: failed to fetch L2 block by hash 0x0000000000000000000000000000000000000000000000000000000000000000
```
 - 这是重新初始化过程：

  - 停止该op-geth过程。
  - 删除geth数据。
```
cd ~/op-geth
rm -rf datadir/geth
```
  - 重新运行 init.

```
build/bin/geth init --datadir=datadir genesis.json
```

  - 运行 op-geth

  - 运行 op-node

- op-node
  一旦开始op-geth运行，就需要运行op-node. 与以太坊一样，OP Stack 有一个共识客户端（op-node）和一个执行客户端（op-geth）。共识客户端通过引擎 API 驱动执行客户端。

```
cd ~/optimism/op-node

./bin/op-node \
	--l2=http://localhost:8551 \
	--l2.jwt-secret=./jwt.txt \    
	--sequencer.enabled \
	--sequencer.l1-confs=3 \
	--verifier.l1-confs=3 \
	--rollup.config=./rollup.json \
	--rpc.addr=0.0.0.0 \
	--rpc.port=8547 \
	--p2p.disable \
	--rpc.enable-admin \
	--p2p.sequencer.key=$SEQ_KEY \
	--l1=$L1_RPC \
	--l1.rpckind=$RPC_KIND

```

- op-batcher

op-batcher从 Sequencer 获取事务并将这些事务发布到 L1 。一旦交易进入 L1，它们就正式成为 Rollup 的一部分。如果没有op-batcher，发送到 Sequencer 的交易将永远不会到达 L1，也不会成为规范链的一部分。这op-batcher很关键！  

最好给予Batcher至少 1 个 Goerli ETH，以确保其能够继续运行而不会耗尽 ETH 的 Gas  

```
cd ~/optimism/op-batcher

./bin/op-batcher \
    --l2-eth-rpc=http://localhost:8545 \
    --rollup-rpc=http://localhost:8547 \
    --poll-interval=1s \
    --sub-safety-margin=6 \
    --num-confirmations=1 \
    --safe-abort-nonce-too-low-count=3 \
    --resubmission-timeout=30s \
    --rpc.addr=0.0.0.0 \
    --rpc.port=8548 \
    --rpc.enable-admin \
    --max-channel-duration=1 \
    --l1-eth-rpc=$L1_RPC \
    --private-key=$BATCHER_KEY
```

- op-proposer

```
cd ~/optimism/op-proposer

./bin/op-proposer \
    --poll-interval=12s \
    --rpc.port=8560 \
    --rollup-rpc=http://localhost:8547 \
    --l2oo-address=$L2OO_ADDR \
    --private-key=$PROPOSER_KEY \
    --l1-eth-rpc=$L1_RPC

```

8. 需要一些 ETH 来支付 Rollup 的 Gas 费
连接钱包后，您可能会注意到 Rollup 上没有任何 ETH。您需要一些 ETH 来支付 Rollup 的 Gas 费。将 Goerli ETH 存入您的链的最简单方法是将资金直接发送到合约L1StandardBridge。L1StandardBridge您可以通过查看包deployments中的文件夹来找到您的链的合约地址contracts-bedrock

- 获取L1标准桥接合约的代理地址
```
cd ~/optimism/packages/contracts-bedrock

cat deployments/getting-started/L1StandardBridgeProxy.json | jq -r .address
```
- 获取 L1 桥接代理合约地址，并使用您希望在 Rollup 上包含 ETH 的钱包，在 Goerli 上向该地址发送少量 ETH（0.1 或更少即可）。该 ETH 最多可能需要 5 分钟才会出现在您的 L2 钱包中。




### 4.Op代码说明
内容章节三的内容在这里。
