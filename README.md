# Nibiru激励性测试网操作说明
    测试网分为四个阶段，第一阶段主要测试Nibiru的Oracle模块和质押，第二阶段主要测试治理与智能合约，第三阶段测试流动性应用Perps&Spot，第四阶段为交易竞赛。目前处于第二阶段，同时第一阶段的质押操作仍可操作。
    参加测试网必须先注册：
    https://gleam.io/yW6Ho/nibiru-incentivized-testnet-registration
    
## 激励性测试网第一阶段-部署节点
    创建验证人（无论是否活跃）有75分。只需要同步一个完整的节点并发送一个创建验证人交易，如果没有被监禁额外加50分

### 安装基础环境-go
    
    
### 下载源代码并编译
    git clone https://github.com/NibiruChain/nibiru
    cd nibiru
    git checkout  v0.19.2
    make install

### 初始化节点
    moniker=<你的节点名>
    nibid init $moniker --chain-id=nibiru-itn-1
    nibid config chain-id nibiru-itn-1

### 下载Genesis文件
    curl -s https://rpc.itn-1.nibiru.fi/genesis | jq -r .result.genesis >  ~/.nibid/config/genesis.json

### 设置peer和seed
   PEERS="df8596fa04abeff1d15b79570ff8c3eba85ed87a@35.185.8.9:26656,4a81486786a7c744691dc500360efcdaf22f0840@15.235.46.50:26656,c709cad9e11b315644fe8f1d2e90c03c5cba685c@34.91.8.241:26656,930b1eb3f0e57b97574ed44cb53b69fb65722786@144.76.30.36:15662,ad002a4592e7bcdfff31eedd8cee7763b39601e7@65.109.122.105:36656"
    seeds="a431d3d1b451629a21799963d9eb10d83e261d2c@seed-1.itn-1.nibiru.fi:26656,6a78a2a5f19c93661a493ecbe69afc72b5c54117@seed-2.itn-1.nibiru.fi:26656"
    sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.nibid/config/config.toml
    sed -i.bak -e "s/^seeds *=.*/seeds = \"$seeds\"/" ~/.nibid/config/config.toml

### Pruning设置
    pruning="custom" && \
    pruning_keep_recent="100" && \
    pruning_keep_every="0" && \
    pruning_interval="10" && \
    sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.nibid/config/app.toml && \
    sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.nibid/config/app.toml && \
    sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.nibid/config/app.toml && \
    sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.nibid/config/app.toml

### 下载addrbook
    wget -O $HOME/.nibid/config/addrbook.json https://snapshot.silentvalidator.com/testnet/nibiru/addrbook.json

### 使用snapshot同步
    sudo apt install lz4
    curl https://snapshots2-testnet.nodejumper.io/nibiru-testnet/nibiru-itn-1_2023-04-19.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.nibid

### 启动节点
    screen -S nibid
    nibid start

### 导入钱包或创建钱包
    各任务必须保证使用的钱包地址统一，如果某钱包已操作过其他任务，则此处应导入钱包，否则新建
    
    新建钱包
    nibid keys add <钱包名>
    
    导入钱包
    nibid keys add <钱包名> --recover
    
### 领取测试币
    方式一：https://app.nibiru.fi/faucet
    上述方式需要安装Keplr钱包：https://chrome.google.com/webstore/detail/keplr/dmkamcknogkgcdfhhbddcghachkejeap?hl=en
    首次访问上述网站需要将nibiru-itn-1网络添加到Keplr钱包
    
    方式二：加入官方DC进入faucet领水，https://github.com/NibiruChain
    $request 钱包地址

### 创建验证人
    nibid tx staking create-validator \
    --amount=1000000unibi \
    --pubkey=$(nibid tendermint show-validator) \
    --moniker=验证人名称 \
    --chain-id=nibiru-itn-1 \
    --commission-rate=0.05 \
    --commission-max-rate=0.2 \
    --commission-max-change-rate=0.1 \
    --min-self-delegation=1 \
    --from=钱包地址 \
    --fees=10000unibi
    
    之后可以去区块浏览器查看验证人是否创建成功。
    https://exp.utsa.tech/nibiru-test/
    https://explorer.kjnodes.com/nibiru-testnet/

## Nibiru激励性测试网第一阶段-质押任务与Oracle模块
    质押者的任务如下,共有100分：
    1、将至少 1 个 NIBI 质押给验证者
    2、领取质押奖励
    3、将你的质押从一个验证者重新委托给另一个验证者
    4、质押给预言机
    5、取消质押您的代币

### 进入区块浏览器操作
    首次进入网站需要边接钱包
    https://explorer.kjnodes.com/nibiru-testnet/









## Nibiru激励性测试网第二阶段-智能合约
    激励测试NIT 2于2023年4月5日开始，主要有两类任务：治理任务、智能合约任务，完成任务可各得300分
    治理任务：参与至少1项治理提案、参与4个或更多治理提案。投票期目前为4天。
    智能合约任务：部署合约、实例化合约、执行合约

### 前提条件
    需要在linux系统服务器安装好nibid程序。可以自己部署节点，也可使用公共的RPC
    
    公共RPC：
    nibid config node https://t-nibiru.rpc.utsa.tech:443
    
    备选的公共RPC：
    https://nibiru-testnet.nodejumper.io:443
    https://rpc-t.nibiru.nodestake.top:443

### 用助记词导入钱包
    nibid keys add xxx --recover
<img width="353" alt="_20230417180254" src="https://user-images.githubusercontent.com/100336530/232453123-d264cf84-5356-4547-be34-9ea52517fea6.png">

### 部署合约
    cd $HOME/.nibid
    mkdir wasm && cd wasm
    wget https://github.com/NibiruChain/cw-nibiru/raw/main/artifacts-cw-plus/cw20_base.wasm
    
    KEY_NAME=钱包名
    CONTRACT_WASM="cw20_base.wasm" 
    nibid tx wasm store $CONTRACT_WASM --from $KEY_NAME --gas=2000000 --fees=200000unibi --chain-id nibiru-itn-1
<img width="795" alt="_20230417180816" src="https://user-images.githubusercontent.com/100336530/232454292-2a335855-5c49-4dc7-a21d-1e76c015c3e8.png">

    txhash=刚刚的txhash
    nibid query tx $txhash --output json | jq -r '.logs[] | .events[]'
<img width="852" alt="_20230417180954" src="https://user-images.githubusercontent.com/100336530/232454712-011f45d7-5dfa-4ed2-baf9-11290d2c7eb5.png">


### 实例化合约
    code_id=上一步查询交易结果所得
    address=你的nibiru钱包地址
    KEY_NAME=你的钱包名
    
    创建一个json文件，用于传递实例化时需要的参数
    tee args.json > /dev/null <<EOF
    {
      "name": "Custom CW20 token",
      "symbol": "CWXX",
      "decimals": 6,
      "initial_balances": [
        {
          "address": "${address}",
          "amount": "555444000"
        }
      ],
      "mint": { "minter": "${address}" },
      "marketing": {}
    }
    EOF

    实例化这个cw20合约
    nibid tx wasm inst $code_id "$(cat args.json)" --label="mint CWXX contract" --no-admin --from=$KEY_NAME --fees=5000unibi --chain-id nibiru-itn-1 -y
<img width="861" alt="_20230417181356" src="https://user-images.githubusercontent.com/100336530/232455604-5919b870-ca1b-4f53-92b6-42224a52fd3c.png">

    获取部署成功得到的合约地址，可通过命令查询：
    txhash=上一步的txhash
    contract=$(nibid q tx $txhash --output json| jq  -r '.logs[] | .events[0] | .attributes[0] | .value')
    echo $contract
![image](https://user-images.githubusercontent.com/100336530/232455708-c77b1ba8-9d4a-4106-bd8b-ef5ee9af8fb8.png)

### 执行合约
    address=任意一个nibiru地址，也可以是自己的地址
    KEY_NAME=你的钱包名
    
    创建一个json文件，用于传递调用合约需要的参数
    tee cw_transfer.json > /dev/null <<EOF
    {
      "transfer": {
        "recipient": "${address}",
        "amount": "50"
      }
    }
    EOF
    
    执行合约
    nibid tx wasm execute $contract "$(cat cw_transfer.json)" --from $KEY_NAME --gas 8000000 --fees=200000unibi -y --chain-id nibiru-itn-1 -y
<img width="855" alt="_20230417181937" src="https://user-images.githubusercontent.com/100336530/232456874-4de344fc-1e8b-42e5-9897-21aa36283ee9.png">

    查询结果
    txhash=刚刚的txhash
    nibid q tx $txhash --output json| jq -r '.logs[] | .events[]'
<img width="839" alt="_20230417182027" src="https://user-images.githubusercontent.com/100336530/232457054-ece8aaa0-6f44-4fdb-a3f8-870312b8cd9e.png">

























