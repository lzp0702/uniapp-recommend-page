<template>
<view class="money-page transfer-page">
  <view class="nav"><text class="back" @tap="back">‹</text><text>转账给 {{name}}</text><text class="more"></text></view>
  <view class="profile"><image src="/static/avatar2.svg" mode="aspectFill"/><view><text class="nickname">{{name}}</text><text class="wxid">微信号：wxid_{{name.toLowerCase().replace(/[^a-z0-9]/g,'').slice(0,8)||'xiaoyu'}}</text></view></view>
  <view class="card">
    <text class="label">转账金额</text>
    <view class="amount-input"><text>¥</text><input :value="amount" type="text" inputmode="none" placeholder="0.00" @input="onInput"/></view>
    <input v-model="note" class="note" maxlength="24" placeholder="添加转账说明（选填）"/>
    <view class="option"><text>到账时间</text><picker :range="arrivalOptions" @change="arrivalIndex=Number($event.detail.value)"><view>{{arrivalOptions[arrivalIndex]}}　›</view></picker></view>
    <view class="option"><text>付款方式</text><picker :range="payments" @change="paymentIndex=Number($event.detail.value)"><view>{{payments[paymentIndex]}}　›</view></picker></view>
    <text class="tip">转账限额以账户及银行卡实际可用额度为准</text>
  </view>
  <view class="keyboard">
    <view class="keys"><view v-for="k in keys" :key="k" @tap="press(k)">{{k}}</view><view @tap="press('0')">0</view><view @tap="press('.')">.</view><view class="del" @tap="press('del')">⌫</view></view>
    <button class="submit" :disabled="!valid" @tap="submit">转账</button>
  </view>
  <text class="demo">前端模拟 · 不连接支付接口，不产生真实资金交易</text>
</view>
</template>
<script setup>
import{ref,computed}from'vue'
import{onLoad}from'@dcloudio/uni-app'
const name=ref('小鱼'),amount=ref(''),note=ref(''),arrivalIndex=ref(0),paymentIndex=ref(0),arrivalOptions=['实时到账','2小时到账','24小时延迟到账'],payments=['零钱（模拟）','银行卡（模拟）'],keys=['1','2','3','4','5','6','7','8','9']
onLoad(q=>{if(q.name)name.value=decodeURIComponent(q.name)})
const valid=computed(()=>{const n=Number(amount.value);return amount.value!==''&&Number.isFinite(n)&&n>=.01&&n<=50000&&/^\d+(\.\d{1,2})?$/.test(amount.value)})
function clean(v){let s=String(v||'').replace(/[^0-9.]/g,'');const i=s.indexOf('.');if(i>=0)s=s.slice(0,i+1)+s.slice(i+1).replace(/\./g,'');if(s.startsWith('.'))s='0'+s;if(s.includes('.')){const p=s.split('.');s=p[0].replace(/^0+(?=\d)/,'')+'.'+p[1].slice(0,2)}else s=s.replace(/^0+(?=\d)/,'');return s}
function onInput(e){amount.value=clean(e.detail.value)}
function press(k){amount.value=k==='del'?amount.value.slice(0,-1):clean(amount.value+k)}
function back(){uni.navigateBack()}
function submit(){if(!valid.value)return;uni.setStorageSync('meet-money-outgoing',{id:Date.now(),kind:'transfer',mine:true,amount:Number(amount.value).toFixed(2),note:note.value.trim(),payment:payments[paymentIndex.value],arrival:arrivalOptions[arrivalIndex.value],status:'pending',createdAt:Date.now(),expiresAt:Date.now()+86400000,claimed:false});uni.navigateBack()}
</script>
<style scoped>
.money-page{position:fixed;inset:0;display:flex;flex-direction:column;box-sizing:border-box;padding-top:env(safe-area-inset-top);background:#f3f3f3;color:#191919;overflow:hidden}.nav{height:48px;display:flex;align-items:center;justify-content:space-between;padding:0 16px;font-size:17px;font-weight:600}.back{font-size:32px;font-weight:300;width:44px}.more{width:44px}.profile{display:flex;align-items:center;gap:12px;margin:26px 18px 20px}.profile image{width:42px;height:42px;border-radius:6px}.profile view{display:flex;flex-direction:column;gap:5px}.nickname{font-size:15px}.wxid{font-size:11px;color:#999}.card{margin:0 16px;background:#fff;border-radius:10px;padding:18px 16px}.label{font-size:14px;color:#555}.amount-input{display:flex;align-items:center;gap:8px;border-bottom:1px solid #eee;height:76px}.amount-input text{font-size:34px;font-weight:600}.amount-input input{flex:1;font-size:38px;font-weight:600;height:68px}.note{height:48px;font-size:14px;border-bottom:1px solid #eee}.option{height:48px;display:flex;align-items:center;justify-content:space-between;font-size:14px;border-bottom:1px solid #f4f4f4}.option picker view{color:#777}.tip{display:block;color:#aaa;font-size:11px;margin-top:13px}.keyboard{margin-top:auto;background:#d1d5db;padding:6px 5px calc(6px + env(safe-area-inset-bottom));display:flex;gap:5px}.keys{flex:3;display:grid;grid-template-columns:repeat(3,1fr);grid-template-rows:repeat(4,49px);gap:4px}.keys view{background:white;border-radius:5px;display:grid;place-items:center;font-size:23px}.keys .del{background:#c5cbd3}.submit{flex:1;margin:0;border:0;border-radius:5px;background:#07c160;color:white;font-size:16px;display:flex;align-items:center;justify-content:center}.submit[disabled]{background:#bfc5c9;color:white}.demo{background:#d1d5db;text-align:center;color:#8c9299;font-size:9px;padding:0 0 4px}
</style>