# Developing-in-Blockchain

Q1: how to attach another geth instance by geth console?

> A: Typing `geth attach` on the terminal.
> The sequence is : 1. Opening  Ethereum wallet 2. Starting  terminal 3. Check  the accounts  weather is the same or not.

## 0x00同步Ethereum wallet 中的网络
Step1：打开Ethereum wallet, 配置栏中选择 Develop-network-solonetwork。
Step2:  创建账户。两种方式： 1.图形化界面中创建。2.console中创建(执行的命令参见下方)。
>personal.newAccount("Password") //password自定义 创建basecoin账户
>personal.newAccount("Password")  //创建第二个账户
>eth.accounts    //显示账户信息和数量

Step3: 挖矿产生一定数量的以太币（ETH）。
打开终端输入：
>miner.start()  //开启挖矿
>miner.stop()   //停止挖矿


当开启挖矿时，wallet中的basecoin账户中以太币的数量就会疯长。


## 0x01在Ethereum私有网络中建立多节点
目录层次说明：


### 搭建节点说明
该测试会在私有网络中搭建两个节点，分别是00，01节点。
genesis.json文件内容：
>{
    "config": {
        "chainId": 15,
        "homesteadBlock": 0,
        "eip155Block": 0,
        "eip158Block": 0
    },
    "coinbase" : "0x0000000000000000000000000000000000000000",
    "difficulty" : "0x40000",
    "extraData" : "",
    "gasLimit" : "0xffffffff",
    "nonce" : "0x0000000000000042",
    "mixhash" : "0x0000000000000000000000000000000000000000000000000000000000000000",
    "parentHash" : "0x0000000000000000000000000000000000000000000000000000000000000000",
    "timestamp" : "0x00",
    "alloc": { }
}


### 1.搭建节点00
#### 初始化创世区块
创建一个文件夹，将上述genesis文件copy其中。
` geth --datadir ./data/00 init genesis.json`


#### 启动节点00
`geth --networkid 14  --nodiscover --datadir ./data/00   --port 61911 --rpcapi net,eth,web3,personal  --rpcport 8101 console`

  
    
    --nodiscover 关闭p2p网络的自动发现，需要手动添加节点，这样有利于我们隐藏私有网络
    --datadir 区块链数据存储目录
    --networkid 网络标识，私有链取一个大于4的随意的值
    --rpc 启用ipc服务，默认端口号8545
    --rpcapi 表示可以通过ipc调用的对象
    --rpcaddr ipc监听地址，默认为127.0.0.1，只能本地访问
    console 打开一个可交互的javascript环境
    更多参数参见：(https://github.com/ethereum/go-ethereum/wiki/Command-Line-Options)



### 2.搭建节点01
#### 初始化创世区块
` geth --datadir ./data/01 init genesis.json`


#### 启动节点01
打开另一个终端，输入：
`geth --networkid 14  --nodiscover --datadir ./data/01   --port 61912 --rpcapi net,eth,web3,personal  --rpcport 8102 --bootnodes "enode://a1163769f3239bfae941ae3e0f7e951e1b31332b80b0ac0897bf3a86d323bcc44de120b399221ef08e8b69b2f22b5f4e15848c8843b0c8733190e0add9e253b9@[::]:61911?discport=0" console
`


  `--botnodes`是设置当前节点启动后,直接通过设置`--botnodes`的值来链接第一个节点, `--botnodes`的值可以通过在第一个节点（00）的命令行中,输入:`admin.nodeInfo.enode`命令打印出来.
  
图为00节点的admin.nodeInfo.enode值：



也可以不设置 `--botnodes`, 直接启动,启动后进入命令行, 通过命令`admin.addPeer(enodeUrlOfFirst Instance)`把它作为一个peer添加进来.


#### 节点的添加
01节点的admin.nodeInfo.enode值：


在00节点中的终端中输入：
`>admin.addPeer("enode://7d1d7752276335e10036be09b189e15079251b4b856a2add8a24dd1ea14ac4a9f6b470db4ba2c47f7e370286f7a19e3b687742b15be4a761cfdfa9f64e2d8bc0@[::]:61912?discport=0")`，将01节点添加进去。
![](/Users/Vinci/Desktop/5.png)



### 3.验证节点

在00节点的终端中输入`net.peerCount`,连接成功的返回值应为1：
>\>net.peerCount
>1
>admin.peers


在01节点的终端中输入`net.peerCount`,连接成功的返回值应为1：

那么到此，我们已经建立了一个包含两个节点的私有网络，按照这样的方式可以建造多节点的本地集群。

## 0X02转账测试
两个节点成功连接后，我们可以继续测试转账的功能。

### 新建账号
00节点：
>\> personal.newAccount("password")
"0x7acff2379fb58f72d6168160ee69cbee882b2e05"


01节点：
> \> personal.newAccount("password")
"0x606c69bb18fa0aaf64b146f9dd2aef4260567d6b"

### 挖矿
01节点：
>\> miner.start(1)//开始挖矿获取一定的Ethers
>\> miner.stop()  //记得停止挖矿
>

### 转账
**01节点：**
> \> personal.unlockAccount(eth.accounts[0],"password") //解锁账户
true
>\>
>eth.sendTransaction({from:"0x606c69bb18fa0aaf64b146f9dd2aef4260567d6b",to:"0x7acff2379fb58f72d6168160ee69cbee882b2e05",value:web3.toWei(10,"ether")}) //转入10个ethers
>
INFO [01-16|13:44:02] Submitted transaction                    fullhash=0x3d0d1d02511f56ba2e5a1dbb1b9e9038f5cd4bca0531b198c0f0b38132488c43 recipient=0x7aCFf2379fb58f72d6168160eE69CBeE882B2E05
"0x3d0d1d02511f56ba2e5a1dbb1b9e9038f5cd4bca0531b198c0f0b38132488c43"
>eth.pendingTransactions

> miner.start(1) //挖矿进行交易验证


**00节点**：
查询当前账户余额：
> \>eth.getBalance(eth.accounts[0])
10000000000000000000
//账户转入10ethers,说明转账成功。

查询块高：
> eth.blockNumber
> 27

