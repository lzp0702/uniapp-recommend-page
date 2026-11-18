<template>
<view class="detail-page packet-detail">
  <view class="nav"><text class="back" @tap="back">‹</text><text>微信红包</text><text class="more"></text></view>
  <view class="content">
    <view class="head"><view class="icon">¥</view><text>{{record.mine?'你发出的红包':(record.chatName||'对方')+'发来的红包'}}</text></view>
    <text class="greeting">{{record.title||'恭喜发财，大吉大利'}}</text>
    <text class="amount" v-if="opened||record.mine||record.claimed">¥{{record.amount||'0.00'}}</text>
    <view v-else class="envelope" @tap="opened=true"><text>拆</text></view>
    <text class="state">{{statusText}}</text>
    <text class="time">{{timeText}}</text>
    <button v-if="!record.mine&&!record.claimed&&record.status!=='expired'&&!opened" class="primary" @tap="opened=true">開</button>
    <button v-if="!record.mine&&!record.claimed&&opened" class="primary" @tap="claim">存入零钱</button>
    <view v-if="record.claimed" class="success">已存入零钱</view>
    <text class="demo">前端交互演示 · 不涉及真实资金</text>
  </view>
</view>
</template>
<script setup>
import{ref,computed}from'vue'
import{onLoad}from'@dcloudio/uni-app'
const record=ref({}),opened=ref(false)
onLoad(()=>{record.value=uni.getStorageSync('meet-money-detail')||{};opened.value=!!record.value.claimed})
const statusText=computed(()=>record.value.status==='claimed'||record.value.claimed?'已领取':record.value.status==='expired'?'已过期，红包已退回':record.value.mine?'等待对方领取':'待领取')
const timeText=computed(()=>record.value.createdAt?new Date(record.value.createdAt).toLocaleString():'')
function save(){const r=record.value;try{const key='meet-chat-'+r.chatName;const list=uni.getStorageSync(key)||[];const i=list.findIndex(x=>x.id===r.id);if(i>=0)list[i]={...list[i],...r};uni.setStorageSync(key,list);uni.setStorageSync('meet-money-detail',r)}catch(e){}}
function claim(){record.value={...record.value,claimed:true,status:'claimed',claimedAt:Date.now()};save();uni.showToast({title:'已存入零钱',icon:'success'})}
function back(){uni.navigateBack()}
</script>
<style scoped>
.detail-page{position:fixed;inset:0;background:#f5f5f5;color:#222;padding-top:env(safe-area-inset-top);box-sizing:border-box}.nav{height:48px;display:flex;align-items:center;justify-content:space-between;padding:0 16px;background:#f7f7f7;font-size:17px;font-weight:600}.back,.more{width:44px;font-size:30px;font-weight:300}.content{margin:12px;background:#fff;border-radius:8px;min-height:75vh;display:flex;flex-direction:column;align-items:center;padding:32px 22px;box-sizing:border-box}.head{display:flex;align-items:center;gap:10px;color:#777;font-size:14px}.icon{width:34px;height:34px;border-radius:50%;background:#f4a13c;color:#fff;display:grid;place-items:center;font-size:21px}.greeting{margin-top:32px;color:#555;font-size:16px}.amount{font-size:42px;font-weight:600;margin-top:24px}.envelope{margin-top:30px;width:116px;height:116px;border-radius:50%;background:#e99b38;color:#fff;display:grid;place-items:center;font-size:42px}.state{font-size:13px;color:#999;margin-top:22px}.time{font-size:11px;color:#bbb;margin-top:9px}.primary{margin-top:28px;width:80%;height:46px;line-height:46px;border:0;border-radius:6px;background:#e99b38;color:#fff;font-size:15px}.success{margin-top:26px;color:#07a65a;font-size:14px}.demo{margin-top:auto;padding-top:36px;color:#bbb;font-size:10px}
</style>