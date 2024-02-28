<template>
  <div class="Web3api">
    <h1>账户信息</h1>
    <van-divider />
    <p><span>账户地址：<b>{{address}}</b></span></p>
    <p><span>账户余额：<b>{{balance}}</b> SepoliaETH</span></p>
    <van-button 
      type="primary"
      loading-type="spinner"
      :loading=loading
      loading-text="转账中..."
      @click="send"
    >
      转账0.01eth
    </van-button>
  </div>
</template>

<script setup>
  import { ref } from 'vue';
  import Web3 from 'web3';
  import Tx from "ethereumjs-tx"
  let address = ref("0x9fd05Ef9CCbEe1795b0343a324B22374aca06fb7");
  let balance = ref(0);
  let loading = ref(false);
  
  const send = async() => {
    loading.value = true;
    // 1、构建转账参数
    // 获取该账号之前发生过多少次交易
    let nonce = await web3.eth.getTransactionCount(address.value);
    console.log("nonce", nonce)
    // 获取当前的gas费用，该值是根据最近几次交易的实际gas费用的平均值
    let gasPrice = await web3.eth.getGasPrice();
    // 设置交易的金额
    let value = web3.utils.toWei("0.01");
    console.log("gasPrice", gasPrice);
    const rawTx = {
      from: address.value,
      to: "0xbCf77fF65102F0f36E18950f1fBB70462d8C547B",
      nonce, // 签署此事务时使用的随机数。默认值将使用 web3.eth.getTransactionCount（）
      gasPrice, // 本次交易设置的 gas 价格，如果为空，则使用 web3.eth.getGasPrice（） 
      value,
      data: "0x0000", // 随便写
    }
    // 2、生成serializerTx
    // 转换私钥, from的私钥
    const privateKey = Buffer.from("626fa4094cc90222e8ed696d321a4da92637460a011aa28b89ad87c40bc43cdd", "hex");
    console.log("privateKey", privateKey)
    // gas 估算
    const gas = await web3.eth.estimateGas(rawTx);
    console.log("gas:", gas)
    rawTx.gas = gas;
    const tx = new Tx(rawTx);
    tx.sign(privateKey);
    var serializedTx = `0x${tx.serialize().toString("hex")}`;
    console.log("serializedTx", serializedTx);
    // 3、开始转账
    const trans = web3.eth.sendSignedTransaction(serializedTx);
    trans.on("transactionHash", (txid) => {
      console.log("交易ID", txid);
      console.log(`https://goerli.etherscan.io/tx/${txid}`);
    });
    trans.on("receipt", (res) => {
      console.log("第一个节点确认：", res);
    });
    trans.on("comfirmation", () => {
      console.log("第n个节点确认");
    });
  }
  // 创建web3实例, 从该网站获取测试网地址：https://app.infura.io/ 
  const web3 = new Web3(Web3.givenProvider || "https://sepolia.infura.io/v3/cc3a9d4927424ee590474244fc2f89d7");
  // 创建账号，并获取地址和私钥，根据私钥可以将账号添加到小狐狸钱包中（谷歌插件）
  //const { address,  privateKey} = web3.eth.accounts.create("123456");
  //console.log("address", address, "privateKey", privateKey);
  // 获取金额
  web3.eth.getBalance(address.value).then(res => {
    // 将位转换为ether
    balance.value = web3.utils.fromWei(res);
  });
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="less">
h1 {
  color: #42b983;
}
</style>
