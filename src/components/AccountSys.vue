<template>
  <p>助记词：{{ mnemonic }}</p>
  <p>种子：{{ seed.toString("hex") }}</p>
  <p>小写地址：{{ lowerCaseAddress }}</p>
  <p>校验地址：{{ checkAddress }}</p>
  <p>私钥：{{ privateKey }}</p>
  <van-button type="primary" @click="exportKStore">导出kStore</van-button>
</template>
<script setup>
  import Web3 from "web3"; 
  import * as bip39 from 'bip39';
  import ethWallet, { hdkey } from "ethereumjs-wallet"; 
  // 创建助记词 soda icon nest amused deal crystal relief scan slogan marriage derive disease
  // const mnemonic = bip39.generateMnemonic();
  const mnemonic = 'soda icon nest amused deal crystal relief scan slogan marriage derive disease';
  // 根据助记词生成密钥对
  const seed = bip39.mnemonicToSeedSync(mnemonic);
  // 分层钱包
  const hdWallet = hdkey.fromMasterSeed(seed);
  // m/purpos'/coin_type'/0'/0/i(钱包的地址)
  const keypair = hdWallet.derivePath("m/44'/60'/0'/0/0");

  console.log("keypair", keypair);
  // 获取私钥
  // 1、获取钱包对象
  const wallet = keypair.getWallet();
  const privateKey = wallet.getPrivateKeyString();
  // 获取地址
  const lowerCaseAddress = wallet.getAddressString();
  // 获取校验地址
  const checkAddress = wallet.getChecksumAddressString();
  /**
   * 导出keystore
   */
  // 方式一、web3.js
  const web3 = new Web3(Web3.givenProvider || "https://sepolia.infura.io/v3/cc3a9d4927424ee590474244fc2f89d7");
  const kStore = web3.eth.accounts.encrypt(privateKey, "123456");
  console.log("kStore", kStore);
  // 方式二, 根据keystore导入账号时需要123456 密码
  async function exportKStore() {
    const kStore = await wallet.toV3("123456");
    console.log("kStore", kStore);
    const jsonData = JSON.stringify(kStore);
    const blob = new Blob([jsonData], { type: "application/json" });
    const fileName = 'Kstore.json';
    const a = document.createElement("a");
    a.href = window.URL.createObjectURL(blob);
    a.download = 'kStore.json';
    a.click();
    //window.URL.revokeObjectURL(a.href);
  }
  /**
   * 根据keystore导入账号获取私钥
   */
  // 方式一：web3.js
  const account1 = web3.eth.accounts.decrypt(kStore, "123456");
  console.log("account1", account1)
  // 方式二：
  ethWallet.fromV3(kStore, "123456").then((wallet) => {
    const account2 = wallet.getPrivateKeyString();
    console.log("account2", account2);
  });
  /**
   * 根据私钥获取地址
   */
  const privateKeyBuffer = Buffer.from(privateKey.slice(2), "hex");
  console.log(privateKeyBuffer)
  const wallet2 = ethWallet.fromPrivateKey(privateKeyBuffer);
  const address = wallet.getAddressString();
  console.log("address", address);
</script>

<style scoped lang="less">

</style>
