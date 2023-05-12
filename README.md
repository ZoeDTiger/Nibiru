# Nibiru激励性测试网操作说明
    测试网分为四个阶段：
    第一阶段主要测试Nibiru的Oracle模块和质押；
    第二阶段主要测试治理与智能合约；
    第三阶段测试流动性应用Perps&Spot；
    第四阶段为交易竞赛。
    
    目前处于第二阶段，同时第一阶段的质押操作仍可操作。
    
    其中质押任务与治理任务普通用户可参与
    部署节点任务、运行oracle、智能合约任务需要linux操作经验
    
    激励测试NIT 2于2023年4月5日开始，主要有两类任务：治理任务、智能合约任务，完成任务可各得300分
    
    参加测试网必须先注册：
    https://gleam.io/yW6Ho/nibiru-incentivized-testnet-registration
    
## 激励性测试网第一阶段-部署节点任务
    创建验证人（无论是否活跃）有75分。只需要同步一个完整的节点并发送一个创建验证人交易，如果没有被监禁额外加50分

##### 安装基础环境-go
    
    
##### 下载源代码并编译
    git clone https://github.com/NibiruChain/nibiru
    cd nibiru
    git checkout  v0.19.2
    make install

##### 初始化节点
    moniker=<你的节点名>
    nibid init $moniker --chain-id=nibiru-itn-1
    nibid config chain-id nibiru-itn-1

##### 下载Genesis文件
    curl -s https://rpc.itn-1.nibiru.fi/genesis | jq -r .result.genesis >  ~/.nibid/config/genesis.json

##### 设置peer和seed
       PEERS="df8596fa04abeff1d15b79570ff8c3eba85ed87a@35.185.8.9:26656,4a81486786a7c744691dc500360efcdaf22f0840@15.235.46.50:26656,c709cad9e11b315644fe8f1d2e90c03c5cba685c@34.91.8.241:26656,930b1eb3f0e57b97574ed44cb53b69fb65722786@144.76.30.36:15662,ad002a4592e7bcdfff31eedd8cee7763b39601e7@65.109.122.105:36656"
    seeds="a431d3d1b451629a21799963d9eb10d83e261d2c@seed-1.itn-1.nibiru.fi:26656,6a78a2a5f19c93661a493ecbe69afc72b5c54117@seed-2.itn-1.nibiru.fi:26656"
    sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.nibid/config/config.toml
    sed -i.bak -e "s/^seeds *=.*/seeds = \"$seeds\"/" ~/.nibid/config/config.toml

##### Pruning设置
    pruning="custom" && \
    pruning_keep_recent="100" && \
    pruning_keep_every="0" && \
    pruning_interval="10" && \
    sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.nibid/config/app.toml && \
    sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.nibid/config/app.toml && \
    sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.nibid/config/app.toml && \
    sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.nibid/config/app.toml

##### 下载addrbook
    wget -O $HOME/.nibid/config/addrbook.json https://snapshot.silentvalidator.com/testnet/nibiru/addrbook.json

##### 使用snapshot同步
    sudo apt install lz4
    curl https://snapshots2-testnet.nodejumper.io/nibiru-testnet/nibiru-itn-1_2023-04-19.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.nibid

##### 启动节点
    screen -S nibid
    nibid start

##### 导入钱包或创建钱包
    各任务必须保证使用的钱包地址统一，如果某钱包已操作过其他任务，则此处应导入钱包，否则新建
    
    新建钱包
    nibid keys add <钱包名>
    
    导入钱包
    nibid keys add <钱包名> --recover
    
##### 领取测试币
    方式一：https://app.nibiru.fi/faucet
    上述方式需要安装Keplr钱包：https://chrome.google.com/webstore/detail/keplr/dmkamcknogkgcdfhhbddcghachkejeap?hl=en
    首次访问上述网站需要将nibiru-itn-1网络添加到Keplr钱包
    
    方式二：加入官方DC进入faucet领水，https://github.com/NibiruChain
    $request 钱包地址

##### 创建验证人
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

## Nibiru激励性测试网第一阶段-质押任务
    质押者的任务如下,共有100分：
    1、将至少 1 个 NIBI 质押给验证者
    2、领取质押奖励
    3、将你的质押从一个验证者重新委托给另一个验证者
    4、质押给预言机
    5、取消质押您的代币

#### 通过区块浏览器操作
    首次进入网站需要连接钱包
    https://explorer.kjnodes.com/nibiru-testnet/
    https://exp.utsa.tech/nibiru-test/
    
##### 连接钱包
<img width="810" alt="_20230419151520" src="https://user-images.githubusercontent.com/100336530/232999929-23aa9ef3-6848-4846-9f3b-74e3b1aa5904.png">
<img width="827" alt="_20230419151541" src="https://user-images.githubusercontent.com/100336530/233000103-0db8cdbe-a32d-4e3d-b979-87ec6dded757.png">
<img width="841" alt="_20230419151627" src="https://user-images.githubusercontent.com/100336530/233000145-576152cc-b7f9-4f54-9d9d-c7d10b27a5cc.png">
<img width="1043" alt="_20230419152728" src="https://user-images.githubusercontent.com/100336530/233000913-de0a4627-8a3e-4a6b-9b84-2eb9e19091f8.png">

##### 开始质押：如果因为gas问题操作失败，可以提高gas费再次质押
<img width="767" alt="_20230419152825" src="https://user-images.githubusercontent.com/100336530/233001141-80cb2556-d222-4564-ab9a-432c07fd378c.png">
<img width="483" alt="_20230419153507" src="https://user-images.githubusercontent.com/100336530/233004398-ceff193b-2ed8-4c4c-95d7-39a88d0da9f7.png">

##### 转质押
<img width="570" alt="_20230419153210" src="https://user-images.githubusercontent.com/100336530/233003947-7089581f-c83c-40e4-8b74-5766bdd50e06.png">
<img width="852" alt="_20230419153224" src="https://user-images.githubusercontent.com/100336530/233003996-fcab223d-ff87-4cc2-8e5a-85fa73417f2c.png">
<img width="856" alt="_20230419153254" src="https://user-images.githubusercontent.com/100336530/233004037-6c1c9a2f-cf96-4139-82f7-d7b232e64cdf.png">
<img width="399" alt="_20230419153415" src="https://user-images.githubusercontent.com/100336530/233004829-8c07b866-5406-445b-8bb2-9caa9f78a6e2.png">

##### 领取质押奖励
<img width="860" alt="_20230419153530" src="https://user-images.githubusercontent.com/100336530/233004959-52930c95-3eb5-4fea-b707-cdd60bded45e.png">

##### 取消质押
<img width="860" alt="_20230419153737" src="https://user-images.githubusercontent.com/100336530/233005077-ef0b8405-aa33-4942-a433-80c8820d79b9.png">

##### 质押给预言机
    同质押操作，选择节点名称带oracle的节点进行质押

## Nibiru激励性测试网第一阶段-运行oracle
    运行oracle ，在超过10000个VotePeriods中投票，并在VotePeriods为所有白名单交易对提供价格能获得200积分。
    
    运行oracle的前提条件是你是一个活跃的验证人（质押前100名验证人）。
    
    注意：预言机是提供价格而不是弃权的验证者。委托给 oracle 算作一个动作的完成。
    

## Nibiru激励性测试网第二阶段-治理任务
    治理任务如下，投票期目前为4天，共300分：
    1、参与至少1项治理提案
    2、参与4个或更多治理提案
    
    目前暂时无提案，不能投票
    
##### 通过区块浏览器投票    
![aPuqAAIcaQpVjybfsCa5d](https://user-images.githubusercontent.com/100336530/233007656-f6e61532-fe81-4715-a2c3-278897c4d416.png)
![38sWwPF-Awgu1hmQPmm9R](https://user-images.githubusercontent.com/100336530/233007801-1a037c05-5449-4f25-94ef-34fd1995e329.png)
![dGjXuLIyzEHX73u-VYM9s](https://user-images.githubusercontent.com/100336530/233007834-e08e4914-e38a-4200-a4bb-8e9b0c031603.png)
![PaLZdFPWr6Tg0QHW4ptIR](https://user-images.githubusercontent.com/100336530/233007849-f2a664d9-4aa8-4e02-a59e-26e21db177e0.png)


## Nibiru激励性测试网第二阶段-智能合约任务
    智能合约任务步骤：部署合约、实例化合约、执行合约
    
##### 前提条件
    需要在linux系统服务器安装好nibid程序。可以自己部署节点，也可使用公共的RPC
    
    公共RPC：
    nibid config node https://t-nibiru.rpc.utsa.tech:443
    
    备选的公共RPC：
    https://nibiru-testnet.nodejumper.io:443
    https://rpc-t.nibiru.nodestake.top:443

##### 用助记词导入钱包
    nibid keys add xxx --recover
<img width="353" alt="_20230417180254" src="https://user-images.githubusercontent.com/100336530/232453123-d264cf84-5356-4547-be34-9ea52517fea6.png">

##### 部署合约
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


##### 实例化合约
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

##### 执行合约
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

##### 快速操作命令
    KEY_NAME=acc_200
    nibid keys add $KEY_NAME --recover --keyring-backend test
    
    sed -i "s/$(cat args.json | jq -r '.name')/$KEY_NAME CW20 token/g" $HOME/.nibid/wasm/args.json
    sed -i "s/$(cat args.json | jq -r '.initial_balances[0].address')/$(nibid keys show $KEY_NAME --keyring-backend test --output json | jq -r '.address')/g" $HOME/.nibid/wasm/args.json
    sed -i "s/$(cat cw_transfer.json | jq -r '.transfer.recipient')/$(nibid keys show $KEY_NAME --keyring-backend test --output json | jq -r '.address')/g" $HOME/.nibid/wasm/cw_transfer.json
    nibid tx wasm store cw20_base.wasm --from $KEY_NAME --gas=2500000 --fees=200000unibi --chain-id nibiru-itn-1 --keyring-backend test --output json -y > store.json
    
    codeid=$(nibid query tx $(cat store.json | jq -r '.txhash') --output json | jq -r '.logs[] | .events[1].attributes[1] | .value')
    echo $codeid
    nibid tx wasm inst $codeid "$(cat args.json)" --label="$(cat store.json | jq -r '.txhash')" --no-admin --from=$KEY_NAME --fees=5000unibi --chain-id nibiru-itn-1 --keyring-backend test --output json -y > inst.json
    
    contract=$(nibid q tx $(cat inst.json | jq -r '.txhash') --output json| jq  -r '.logs[] | .events[0] | .attributes[0] | .value')
    echo $contract
    nibid tx wasm execute $contract "$(cat cw_transfer.json)" --from $KEY_NAME --gas 2500000 --fees=200000unibi -y --chain-id nibiru-itn-1 --keyring-backend test --output json -y > execute.json























