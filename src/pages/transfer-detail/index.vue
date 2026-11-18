<template>
<view class="detail-page">
  <view class="nav"><text class="back" @tap="back">‹</text><text>转账详情</text><text class="more"></text></view>
  <view class="content">
    <view class="icon">↗</view>
    <text class="amount">¥{{record.amount||'0.00'}}</text>
    <text class="note">{{record.note||'转账'}}</text>
    <text class="status">{{statusText}}</text>
    <view class="line"></view>
    <view class="info"><text>收款方</text><text>{{record.chatName||'对方'}}</text></view>
    <view class="info"><text>转账时间</text><text>{{timeText}}</text></view>
    <view class="info"><text>到账时间</text><text>{{record.arrival||'实时到账'}}</text></view>
    <view class="info"><text>付款方式</text><text>{{record.payment||'零钱（模拟）'}}</text></view>
    <button v-if="!record.mine&&record.status==='pending'" class="primary" @tap="accept">确认收款</button>
    <button v-if="!record.mine&&record.status==='pending'" class="secondary" @tap="refund">退还</button>
    <text class="demo">前端交互演示 · 不涉及真实资金</text>
  </view>
</view>
</template>
<script setup>
import{ref,computed}from'vue'
import{onLoad}from'@dcloudio/uni-app'
const record=ref({})
onLoad(()=>{record.value=uni.getStorageSync('meet-money-detail')||{}})
const statusText=computed(()=>record.value.status==='claimed'?'已收款':record.value.status==='returned'?'已退还':record.value.mine?'等待对方确认收款':'待确认收款')
const timeText=computed(()=>record.value.createdAt?new Date(record.value.createdAt).toLocaleString():'')
function save(){const r=record.value;try{const key='meet-chat-'+r.chatName;const list=uni.getStorageSync(key)||[];const i=list.findIndex(x=>x.id===r.id);if(i>=0)list[i]={...list[i],...r};uni.setStorageSync(key,list);uni.setStorageSync('meet-money-detail',r)}catch(e){}}
function accept(){record.value={...record.value,status:'claimed',claimed:true,claimedAt:Date.now()};save();uni.showToast({title:'已确认收款',icon:'success'})}
function refund(){record.value={...record.value,status:'returned',returnedAt:Date.now()};save();uni.showToast({title:'已退还',icon:'success'})}
function back(){uni.navigateBack()}
</script>
<style scoped>
.detail-page{position:fixed;inset:0;background:#f5f5f5;color:#222;padding-top:env(safe-area-inset-top);box-sizing:border-box}.nav{height:48px;display:flex;align-items:center;justify-content:space-between;padding:0 16px;background:#f7f7f7;font-size:17px;font-weight:600}.back,.more{width:44px;font-size:30px;font-weight:300}.content{margin:12px;background:#fff;border-radius:8px;min-height:75vh;padding:35px 22px;box-sizing:border-box;display:flex;flex-direction:column;align-items:center}.icon{width:48px;height:48px;border-radius:50%;background:#f2b84e;color:#fff;display:grid;place-items:center;font-size:28px}.amount{font-size:42px;font-weight:600;margin-top:20px}.note{font-size:14px;color:#666;margin-top:8px}.status{font-size:13px;color:#999;margin-top:22px}.line{width:100%;height:1px;background:#eee;margin:32px 0 12px}.info{width:100%;display:flex;justify-content:space-between;align-items:center;min-height:43px;font-size:13px}.info text:first-child{color:#999}.info text:last-child{color:#333}.primary,.secondary{width:100%;height:45px;line-height:45px;border:0;border-radius:6px;margin-top:14px;font-size:14px}.primary{background:#07c160;color:#fff}.secondary{background:#f5f5f5;color:#555}.demo{margin-top:auto;padding-top:30px;color:#bbb;font-size:10px}
</style>