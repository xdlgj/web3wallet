<template>
  <div class="Web3api">
    <van-button type="primary">Web3api</van-button>
  </div>
</template>

<script setup>
  import Web3 from 'web3';
  // 创建web3实例
  const web3 = new Web3(Web3.givenProvider || "https://goerli.infura.io/v3/cc3a9d4927424ee590474244fc2f89d7");
  // 创建账号，并获取地址
  const { address } = web3.eth.accounts.create("123456");
  console.log("address", address);
  // 获取金额，setup函数中可以直接使用await
  web3.eth.getBalance(address).then(res => {
    console.log("balance", res)
  });
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="less">
</style>
