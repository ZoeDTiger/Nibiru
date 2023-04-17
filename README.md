# Nibiru激励性测试网第二阶段-智能合约
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

























