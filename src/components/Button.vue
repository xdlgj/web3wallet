<template>
  <van-overlay :show="loading" style="display: flex; align-items: center; justify-content: center;">
    <van-loading color="#1989fa" :show="true">创建中...</van-loading>
  </van-overlay>
  <van-list
    :loading="false"
    finished-text="没有更多了"
  >
    <van-cell v-for="wallet in wallets" :key="wallet.id">
      <div style="display: flex;">
        <span>账号{{wallet.id}}:</span>
        <van-text-ellipsis :content="wallet.address" position="middle" style="width: 80%;" />
      </div>
    </van-cell>
  </van-list>
  <template v-if="mnemonic != ''">
    <span>{{mnemonic}}</span>
    <van-icon name="description-o" @click="copyMnemonic"/>
  </template>
  <van-row type="flex" justify="space-around">
    <van-button type="primary" @click="createAccount">创建钱包</van-button>
    <van-button type="primary">导入钱包</van-button>
  </van-row>
  <van-dialog 
    :show="showDialog" 
    title="请输入密码" 
    show-cancel-button 
    @cancel="() => {showDialog = false}"
    @confirm="btnConfirm"
  >
    <van-field
      type="password"
      v-model="password"
      autosize
      autofocus
      maxlength="50"
      placeholder="请输入密码"
      show-word-limit
    />
  </van-dialog>
  <van-dialog
    type="textarea"
    :show="showDialogMn" 
    title="请输助记词" 
    show-cancel-button 
    @cancel="() => {showDialogMn = false}"
    @confirm="btnConfirm2"
  >
    <van-field
      v-model="mnemonicRepl"
      autosize
      autofocus
      maxlength="200"
      placeholder="请输入助记词"
      show-word-limit
    />
  </van-dialog>
</template>
<script setup>
import { ref, reactive } from "vue";
import { showNotify } from 'vant';
import 'vant/es/notify/style';
import * as bip39 from 'bip39';
import { hdkey } from "ethereumjs-wallet";
import store from 'store2';

const loading = ref(false);
const showDialog = ref(false);
const showDialogMn = ref(false);
const password = ref("");
const mnemonic = ref("");
const mnemonicRepl = ref(""); // 用户确认助记词
const wallets = reactive(store.get("wallets") || []);
const walletIdx = ref(store.get("wallets") ? store.get("wallets").length : 0);

function createAccount() {
  showDialog.value = true;
}
function btnConfirm() {
  if (password.value.trim()) {
    showDialog.value = false;
    mnemonic.value = wallets.length != 0 ? wallets[0].mnemonic : bip39.generateMnemonic();
  } else {
    showNotify({ type: 'warning', message: '密码不能为空!!' });
  }
}
function btnConfirm2() {
  if (mnemonic.value.trim() == mnemonicRepl.value.trim()) {
    showDialogMn.value = false;
    createWallet();
  } else {
    showNotify({ type: 'warning', message: '助记词错误!!' });
  }
}
// 拷贝助记词
function copyMnemonic() {
  navigator.clipboard.writeText(mnemonic.value);
  showNotify({ type: 'success', message: '助记词已复制到剪贴板' });
  showDialogMn.value = true;
}
// 生成钱包
async function createWallet() {
  loading.value = true;
  const seedBuf = bip39.mnemonicToSeedSync(mnemonic.value);
  let ethereumHDKey = hdkey.fromMasterSeed(seedBuf);
  ethereumHDKey = ethereumHDKey.derivePath(`m/44'/60'/0'/0/${walletIdx.value}`);
  const wallet = ethereumHDKey.getWallet();
  walletIdx.value = walletIdx.value + 1;
  const walletInfo = {id: walletIdx.value};
  walletInfo.address = wallet.getAddressString();
  walletInfo.privateKey = wallet.getPrivateKeyString();
  walletInfo.publicKey = wallet.getPublicKeyString();
  walletInfo.kstore = await wallet.toV3(password.value);
  walletInfo.password = password.value;
  walletInfo.mnemonic = mnemonic.value;
  wallets.push(walletInfo);
  console.log("wallets", wallets);
  store("wallets", wallets);
  loading.value = false;
  showNotify({ type: 'success', message: '创建成功' });
}

</script>

<style scoped lang="less">
</style>