<template>
<view class="money-page packet-page">
  <view class="nav"><text class="back" @tap="back">‹</text><text>发红包</text><text class="more" @tap="showInfo">···</text></view>
  <view class="form-list">
    <view class="row"><text>金额</text><view class="amount-input"><text>¥</text><input :value="amount" type="text" inputmode="none" placeholder="0.00" @input="onInput"/></view></view>
    <view class="greeting"><input v-model="greeting" maxlength="24" placeholder="恭喜发财，大吉大利"/><text @tap="addEmoji">☺＋</text></view>
    <view class="row" @tap="chooseCover"><text>红包封面</text><text class="muted">{{covers[coverIndex]}}　›</text></view>
  </view>
  <view class="preview">¥{{amount||'0.00'}}</view>
  <button class="send" :disabled="!valid" @tap="submit">塞钱进红包</button>
  <text class="wallet-tip">可直接使用收到的零钱发红包</text>
  <view class="keyboard">
    <view class="keys"><view v-for="k in keys" :key="k" @tap="press(k)">{{k}}</view><view @tap="press('0')">0</view><view @tap="press('.')">.</view><view class="del" @tap="press('del')">⌫</view></view>
    <button class="confirm" :disabled="!valid" @tap="submit">确定</button>
  </view>
  <text class="demo">前端模拟 · 不连接支付接口，不产生真实资金交易</text>
</view>
</template>
<script setup>
import{ref,computed}from'vue'
import{onLoad}from'@dcloudio/uni-app'
const name=ref('小鱼'),amount=ref(''),greeting=ref(''),coverIndex=ref(0),covers=['经典红','喜庆金','节日限定'],keys=['1','2','3','4','5','6','7','8','9']
onLoad(q=>{if(q.name)name.value=decodeURIComponent(q.name)})
const valid=computed(()=>{const n=Number(amount.value);return amount.value!==''&&Number.isFinite(n)&&n>=.01&&n<=200&&/^\d+(\.\d{1,2})?$/.test(amount.value)})
function clean(v){let s=String(v||'').replace(/[^0-9.]/g,'');const i=s.indexOf('.');if(i>=0)s=s.slice(0,i+1)+s.slice(i+1).replace(/\./g,'');if(s.startsWith('.'))s='0'+s;if(s.includes('.')){const p=s.split('.');s=p[0].replace(/^0+(?=\d)/,'')+'.'+p[1].slice(0,2)}else s=s.replace(/^0+(?=\d)/,'');return s}
function onInput(e){amount.value=clean(e.detail.value)}
function press(k){amount.value=k==='del'?amount.value.slice(0,-1):clean(amount.value+k)}
function back(){uni.navigateBack()}
function chooseCover(){uni.showActionSheet({itemList:covers,success:r=>coverIndex.value=r.tapIndex})}
function addEmoji(){const e=['😊','❤️','🎉','🌹','🥰'];greeting.value=(greeting.value||'')+e[Math.floor(Math.random()*e.length)]}
function showInfo(){uni.showActionSheet({itemList:['红包说明','取消'],success:r=>{if(r.tapIndex===0)uni.showToast({title:'单个红包金额 0.01～200 元',icon:'none'})}})}
function submit(){if(!valid.value)return;uni.setStorageSync('meet-money-outgoing',{id:Date.now(),kind:'redpacket',mine:true,amount:Number(amount.value).toFixed(2),title:greeting.value.trim()||'恭喜发财，大吉大利',cover:covers[coverIndex.value],status:'pending',createdAt:Date.now(),expiresAt:Date.now()+86400000,claimed:false});uni.navigateBack()}
</script>
<style scoped>
.money-page{position:fixed;inset:0;display:flex;flex-direction:column;box-sizing:border-box;padding-top:env(safe-area-inset-top);background:#f3f3f3;color:#191919;overflow:hidden}.nav{height:48px;display:flex;align-items:center;justify-content:space-between;padding:0 16px;font-size:17px;font-weight:600}.back{font-size:32px;font-weight:300;width:44px}.more{font-size:25px;width:44px;text-align:right}.form-list{margin:28px 16px 0;background:#fff;border-radius:10px;padding:0 16px}.row{height:68px;display:flex;align-items:center;justify-content:space-between;border-bottom:1px solid #f1f1f1;font-size:16px}.amount-input{display:flex;align-items:center;gap:8px}.amount-input text{font-size:22px;font-weight:600}.amount-input input{width:190px;text-align:right;font-size:25px;font-weight:600;height:50px}.greeting{height:60px;display:flex;align-items:center;gap:12px;border-bottom:1px solid #f1f1f1}.greeting input{flex:1;font-size:15px}.greeting text{font-size:22px;color:#999}.muted{color:#999}.preview{text-align:center;font-size:42px;font-weight:650;margin-top:56px}.send{margin:28px 42px 0;background:#f0a23a;color:white;border-radius:7px;font-size:17px;height:48px;line-height:48px}.send[disabled],.confirm[disabled]{background:#d8d8d8;color:#fff}.wallet-tip{text-align:center;color:#999;font-size:12px;margin-top:18px}.keyboard{margin-top:auto;background:#d1d5db;padding:6px 5px calc(6px + env(safe-area-inset-bottom));display:flex;gap:5px}.keys{flex:3;display:grid;grid-template-columns:repeat(3,1fr);grid-template-rows:repeat(4,49px);gap:4px}.keys view{background:white;border-radius:5px;display:grid;place-items:center;font-size:23px}.keys .del{background:#c5cbd3}.confirm{flex:1;margin:0;border:0;border-radius:5px;background:#f0a23a;color:white;font-size:16px;display:flex;align-items:center;justify-content:center}.demo{background:#d1d5db;text-align:center;color:#8c9299;font-size:9px;padding:0 0 4px}
</style>