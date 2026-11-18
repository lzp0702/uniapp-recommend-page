<template>
<view class="chat-page">
  <view class="topbar"><text class="back" @tap="back">‹</text><view class="top-title"><text class="title">{{name}}</text><text class="online">{{peerStatus}}</text></view><text class="more" @tap="chatMenu">···</text></view>
  <scroll-view class="message-scroll" scroll-y :scroll-into-view="lastMessageId" :scroll-with-animation="true">
    <view class="history-hint" @tap="toast='暂时没有更多历史消息'">— 以上是打招呼的内容 —</view>
    <view class="time-label">今天 15:28</view>
    <view v-for="(m,i) in list" :key="m.id||i" :id="'msg-'+i" class="message-row" :class="{mine:m.mine}">
      <image class="avatar" :src="m.mine?myAvatar:peerAvatar" mode="aspectFill"/>
      <view class="message-content">
        <view v-if="m.kind==='image'" class="image-bubble" @tap="previewImage(m.text)"><image :src="m.text" mode="widthFix"/></view>
        <view v-else-if="m.kind==='voice'" class="voice-bubble" @tap="toast='语音消息播放需接入音频服务'"><text>◖</text><text>{{m.duration||'6'}}″</text></view>
        <view v-else class="bubble">{{m.text}}</view>
        <text class="message-state" v-if="m.mine">已发送</text>
      </view>
    </view>
    <view id="message-bottom" class="bottom-anchor"></view>
  </scroll-view>
  <view v-if="showTools" class="tool-panel">
    <view class="tool-item" @tap="choosePhoto"><view class="tool-icon photo">▧</view><text>照片</text></view>
    <view class="tool-item" @tap="takePhoto"><view class="tool-icon camera">◉</view><text>拍摄</text></view>
    <view class="tool-item" @tap="sendLocation"><view class="tool-icon location">⌖</view><text>位置</text></view>
    <view class="tool-item" @tap="sendContact"><view class="tool-icon contact">♙</view><text>名片</text></view>
    <view class="tool-item" @tap="toast='红包功能需接入支付服务'"><view class="tool-icon redpacket">￥</view><text>红包</text></view>
    <view class="tool-item" @tap="toast='视频通话需接入实时音视频服务'"><view class="tool-icon video">▻</view><text>视频通话</text></view>
  </view>
  <view v-if="showEmoji" class="emoji-panel"><text v-for="e in emojis" :key="e" @tap="draft+=e">{{e}}</text></view>
  <view class="composer">
    <view class="voice-toggle" @tap="voiceMode=!voiceMode">{{voiceMode?'⌨':'◖'}}</view>
    <view v-if="voiceMode" class="hold-talk" @touchstart="talking=true" @touchend="finishTalk" @touchcancel="finishTalk">{{talking?'松开发送':'按住 说话'}}</view>
    <input v-else v-model="draft" class="message-input" confirm-type="send" placeholder="发送消息" @confirm="send"/>
    <view class="emoji-button" @tap="showEmoji=!showEmoji;showTools=false">☺</view>
    <view v-if="!draft.trim()" class="plus-button" @tap="showTools=!showTools;showEmoji=false">＋</view>
    <view v-else class="send-button" @tap="send">发送</view>
  </view>
  <view v-if="toast" class="toast" @tap="toast=''">{{toast}}</view>
</view>
</template>
<script setup>
import{ref,nextTick}from'vue'
import{onLoad,onShow}from'@dcloudio/uni-app'
const name=ref('小鱼'),draft=ref(''),toast=ref(''),showTools=ref(false),showEmoji=ref(false),voiceMode=ref(false),talking=ref(false),peerStatus=ref('对方在线'),lastMessageId=ref(''),myAvatar=ref('/static/avatar1.svg'),peerAvatar=ref('/static/avatar2.svg')
const emojis=['😀','😂','🥰','😍','😊','😉','😘','🤔','👍','❤️','🎉','☀️','🌹','🙏','😭','😎','🐱','🍀','✨','💗']
const list=ref([{id:1,text:'嗨，刚刚看到你的动态～',mine:false},{id:2,text:'你好呀，很高兴认识你',mine:true},{id:3,text:'你也喜欢旅行吗？',mine:false}])
function storageKey(){return'meet-chat-'+name.value}
function loadMessages(){try{const saved=uni.getStorageSync(storageKey());if(saved)list.value=saved}catch(e){}}
onLoad((q={})=>{if(q.name)name.value=decodeURIComponent(q.name);loadMessages();scrollBottom()})
onShow(()=>loadMessages())
function persist(){uni.setStorageSync(storageKey(),list.value);scrollBottom()}
function scrollBottom(){nextTick(()=>{lastMessageId.value='';nextTick(()=>{lastMessageId.value='message-bottom'})})}
function send(){const value=draft.value.trim();if(!value)return;if(list.value.length>=500){toast.value='聊天记录太多，请清理后再发送';return}list.value.push({id:Date.now(),text:value,mine:true,time:Date.now()});draft.value='';showTools.value=false;showEmoji.value=false;persist()}
function choosePhoto(){uni.chooseImage({count:9,sizeType:['compressed'],success:r=>{r.tempFilePaths.forEach((src,i)=>list.value.push({id:Date.now()+i,text:src,mine:true,kind:'image'}));persist();showTools.value=false}})}
function takePhoto(){uni.chooseImage({count:1,sourceType:['camera'],sizeType:['compressed'],success:r=>{list.value.push({id:Date.now(),text:r.tempFilePaths[0],mine:true,kind:'image'});persist();showTools.value=false},fail:()=>{toast.value='无法打开相机，请检查系统权限'}})}
function previewImage(src){const urls=list.value.filter(x=>x.kind==='image').map(x=>x.text);uni.previewImage({current:src,urls})}
function sendLocation(){toast.value='位置分享需要接入地图定位服务'}
function sendContact(){uni.showModal({title:'发送个人名片',content:'确认将自己的个人主页分享给对方？',success:r=>{if(r.confirm){list.value.push({id:Date.now(),text:'分享了个人主页',mine:true});persist()}}});showTools.value=false}
function finishTalk(){if(talking.value){talking.value=false;toast.value='语音消息录制需接入录音与上传服务'}}
function chatMenu(){uni.showActionSheet({itemList:['查看对方主页','搜索聊天记录','清空聊天记录'],success:r=>{if(r.tapIndex===0)uni.navigateTo({url:'/pages/user/index?name='+encodeURIComponent(name.value)});if(r.tapIndex===1)toast.value='聊天记录搜索：可在本地记录中查找';if(r.tapIndex===2)uni.showModal({title:'清空聊天记录',content:'确定删除本机保存的聊天记录吗？',success:x=>{if(x.confirm){list.value=[];uni.removeStorageSync(storageKey())}}})}})}
function back(){uni.navigateBack()}
</script>
<style scoped lang="scss">
.chat-page{height:100vh;display:flex;flex-direction:column;background:#ededed;color:#191919;overflow:hidden}.topbar{height:54px;padding:0 14px;display:flex;align-items:center;justify-content:space-between;background:#f7f7f7;border-bottom:1px solid #e8e8e8;flex-shrink:0}.back{font-size:31px;width:38px;color:#222;line-height:1}.top-title{flex:1;text-align:center}.title{display:block;font-size:16px;font-weight:650}.online{display:block;font-size:9px;color:#999;margin-top:3px}.more{width:38px;text-align:right;font-size:25px;letter-spacing:1px}.message-scroll{flex:1;min-height:0;padding:12px 13px;box-sizing:border-box}.history-hint{text-align:center;color:#b2b2b2;font-size:10px;margin:9px 0 16px}.time-label{text-align:center;font-size:10px;color:#aaa;margin:18px 0}.message-row{display:flex;align-items:flex-start;gap:9px;margin:16px 0}.message-row.mine{flex-direction:row-reverse}.avatar{width:39px;height:39px;border-radius:5px;background:#d9d9d9;flex-shrink:0}.message-content{max-width:72%;display:flex;flex-direction:column}.bubble{padding:10px 12px;border-radius:4px 5px 5px 5px;background:#fff;font-size:14px;line-height:1.55;word-break:break-word;position:relative}.mine .bubble{background:#95ec69;border-radius:5px 4px 5px 5px}.message-state{font-size:9px;color:#aaa;margin-top:4px}.mine .message-state{text-align:right}.image-bubble image{display:block;max-width:190px;max-height:230px;border-radius:4px}.voice-bubble{min-width:92px;padding:10px 12px;background:#fff;border-radius:5px;display:flex;justify-content:space-between;font-size:14px}.mine .voice-bubble{background:#95ec69}.bottom-anchor{height:2px}.tool-panel{background:#f6f6f6;border-top:1px solid #e5e5e5;padding:18px 14px 22px;display:grid;grid-template-columns:repeat(4,1fr);gap:20px 8px}.tool-item{display:flex;flex-direction:column;align-items:center;gap:7px;font-size:10px;color:#666}.tool-icon{width:48px;height:48px;border-radius:12px;background:#fff;display:grid;place-items:center;font-size:24px;color:#555}.tool-icon.redpacket{color:#e96b55}.tool-icon.video{color:#72a9c9}.emoji-panel{background:#f6f6f6;padding:12px;display:grid;grid-template-columns:repeat(7,1fr);gap:12px;text-align:center;font-size:23px}.composer{min-height:56px;background:#f7f7f7;border-top:1px solid #e4e4e4;display:flex;align-items:center;gap:9px;padding:7px 10px calc(7px + env(safe-area-inset-bottom));box-sizing:content-box;flex-shrink:0}.voice-toggle,.emoji-button,.plus-button{font-size:25px;width:30px;text-align:center;color:#333;flex-shrink:0}.message-input{height:39px;min-width:0;flex:1;background:#fff;border-radius:5px;padding:0 10px;font-size:14px}.hold-talk{height:39px;flex:1;background:#fff;border-radius:5px;display:grid;place-items:center;font-size:14px;color:#333}.send-button{background:#07c160;color:#fff;border-radius:5px;padding:9px 12px;font-size:12px;flex-shrink:0}.toast{position:fixed;z-index:30;bottom:90px;left:10%;width:80%;text-align:center;background:#333e;color:#fff;padding:11px;border-radius:7px;font-size:12px}
</style>