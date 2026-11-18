<template>
<view class="chat-page" :style="{background:chatBg}">
  <view class="topbar"><text class="back" @tap="back">‹</text><view class="top-title"><text class="title">{{name}}</text><text class="online">{{peerStatus}}</text></view><text class="more" @tap="chatMenu">···</text></view>
  <scroll-view class="message-scroll" scroll-y :scroll-into-view="lastMessageId" :scroll-with-animation="true">
    <view class="history-hint" @tap="toast='暂时没有更多历史消息'">— 以上是打招呼的内容 —</view>
    <view class="time-label">今天 15:28</view>
    <view v-for="(m,i) in list" :key="m.id||i" :id="'msg-'+i" class="message-row" :class="{mine:m.mine}" @longpress="messageActions(m,i)">
      <image class="avatar" :src="m.mine?myAvatar:peerAvatar" mode="aspectFill"/>
      <view class="message-content">
        <view v-if="m.kind==='image'" class="image-bubble" @tap="previewImage(m.text)"><image :src="m.text" mode="widthFix"/></view>
        <view v-else-if="m.kind==='redpacket'" class="redpacket-message" @tap="openPacket(m)"><view class="packet-top"><text class="packet-symbol">￥</text><view><b>{{m.title||'恭喜发财，大吉大利'}}</b><small>{{m.claimed?'已领取':'恭喜发财'}}</small></view></view><view class="packet-bottom">遇见红包</view></view>
        <view v-else-if="m.kind==='transfer'" class="transfer-message"><view class="transfer-top"><text>￥</text><view><b>¥{{m.amount}}</b><small>{{m.claimed?'已收款':'转账'}}</small></view></view><view class="transfer-bottom">微信式转账 · 演示记录</view></view>
        <view v-else class="bubble">{{m.text}}</view>
        <text class="message-state" v-if="m.mine">已发送</text>
      </view>
    </view>
    <view id="message-bottom" class="bottom-anchor"></view>
  </scroll-view>
  <view v-if="showTools" class="tool-panel">
    <view class="tool-item" @tap="choosePhoto"><view class="tool-icon photo">▧</view><text>相册</text></view>
    <view class="tool-item" @tap="takePhoto"><view class="tool-icon camera">◉</view><text>拍摄</text></view>
    <view class="tool-item" @tap="chooseFile"><view class="tool-icon file">▤</view><text>文件</text></view>
    <view class="tool-item" @tap="sendLocation"><view class="tool-icon location">⌖</view><text>位置</text></view>
    <view class="tool-item" @tap="openPacketForm"><view class="tool-icon redpacket">￥</view><text>红包</text></view>
    <view class="tool-item" @tap="openTransferForm"><view class="tool-icon transfer">↗</view><text>转账</text></view>
    <view class="tool-item" @tap="sendContact"><view class="tool-icon contact">♙</view><text>个人名片</text></view>
    <view class="tool-item" @tap="sendFavorite"><view class="tool-icon favorite">☆</view><text>收藏</text></view>
  </view>
  <view v-if="packetModal" class="modal-mask" @tap="packetModal=false"><view class="money-modal packet-modal" @tap.stop><view class="money-head">发红包</view><view class="packet-type"><text :class="{active:packetType==='normal'}" @tap="packetType='normal'">普通红包</text><text :class="{active:packetType==='lucky'}" @tap="packetType='lucky'">拼手气红包</text></view><view class="money-label">金额</view><view class="money-input"><text>¥</text><input v-model="moneyAmount" type="digit" placeholder="0.00"/></view><view class="money-note"><input v-model="moneyGreeting" maxlength="24" placeholder="恭喜发财，大吉大利"/></view><view class="money-explain">当前为交互演示，不会扣款或产生真实资金交易</view><button class="money-submit" @tap="sendPacket">塞钱进红包</button><text class="money-cancel" @tap="packetModal=false">取消</text></view></view>
  <view v-if="transferModal" class="modal-mask" @tap="transferModal=false"><view class="money-modal" @tap.stop><view class="money-head">转账给 {{name}}</view><view class="money-label">转账金额</view><view class="money-input"><text>¥</text><input v-model="moneyAmount" type="digit" placeholder="0.00"/></view><view class="money-note"><input v-model="moneyGreeting" maxlength="24" placeholder="添加转账说明（选填）"/></view><view class="money-explain">演示转账仅生成聊天记录，不涉及支付</view><button class="money-submit transfer-submit" @tap="sendTransfer">确认转账</button><text class="money-cancel" @tap="transferModal=false">取消</text></view></view>
  <view v-if="showEmoji" class="emoji-panel">
    <view class="emoji-grabber" @tap="emojiExpanded=!emojiExpanded"><view></view></view>
    <view class="emoji-tabs"><text :class="{selected:emojiTab==='recent'}" @tap="emojiTab='recent'">◷</text><text :class="{selected:emojiTab==='emoji'}" @tap="emojiTab='emoji'">☺</text><text :class="{selected:emojiTab==='stickers'}" @tap="emojiTab='stickers'">▧</text><text class="emoji-manage" @tap="toast='表情管理入口：可在正式版本接入表情商店'">＋</text></view>
    <scroll-view class="emoji-grid" scroll-y :class="{expanded:emojiExpanded}">
      <view v-if="emojiTab==='recent'||emojiTab==='emoji'" class="emoji-cells"><text v-for="e in (emojiTab==='recent'?recentEmojis:emojis)" :key="e" @tap="insertEmoji(e)">{{e}}</text></view>
      <view v-else class="sticker-placeholder"><text>☺</text><small>暂无添加的表情包</small><button @tap="toast='表情商店需要接入表情资源'">添加表情</button></view>
    </scroll-view>
    <view class="emoji-footer"><text @tap="draft=draft.slice(0,-2)">删除</text><button :disabled="!draft.trim()" @tap="send">发送</button></view>
  </view>
  <view class="composer" :class="{'panel-open':showEmoji||showTools}">
    <textarea v-model="draft" class="message-input" auto-height maxlength="-1" :show-confirm-bar="false" cursor-spacing="8" placeholder="发送消息" @confirm="send"/>
    <view class="emoji-button" @tap="showEmoji=!showEmoji;showTools=false">☺</view>
    <view v-if="!draft.trim()" class="plus-button" @tap="showTools=!showTools;showEmoji=false">＋</view>
    <view v-else class="send-button" @tap="send">发送</view>
  </view>
  <view v-if="settingsVisible" class="sheet-mask" @tap="settingsVisible=false"><view class="settings-sheet" @tap.stop><view class="sheet-grabber"></view><view class="sheet-title">聊天设置</view><view class="settings-list">
    <view class="settings-item" @tap="toggleMute"><text>消息免打扰</text><switch :checked="muted" color="#07c160" @change="toggleMute"/></view>
    <view class="settings-item" @tap="toggleTop"><text>置顶聊天</text><switch :checked="pinned" color="#07c160" @change="toggleTop"/></view>
    <view class="settings-item"><text>提醒</text><switch :checked="true" color="#07c160" disabled/></view>
    <view class="settings-gap"></view>
    <view class="settings-item arrow" @tap="changeBackground"><text>设置当前聊天背景</text><text>›</text></view>
    <view class="settings-item arrow" @tap="searchHistory"><text>查找聊天记录</text><text>›</text></view>
    <view class="settings-item arrow" @tap="clearHistory"><text>清空聊天记录</text><text>›</text></view>
    <view class="settings-gap"></view>
    <view class="settings-item arrow" @tap="settingsVisible=false"><text>投诉</text><text>›</text></view>
    <view class="settings-item arrow" @tap="settingsVisible=false"><text>取消</text><text>›</text></view>
  </view></view></view>
  <view v-if="searchModal" class="search-mask"><view class="search-box"><view class="search-title">查找聊天记录</view><view class="search-input"><input v-model="searchKeyword" placeholder="搜索聊天内容" confirm-type="search"/><text @tap="searchKeyword=''">清除</text></view><view v-if="searchKeyword" class="search-results"><view v-for="(m,i) in searchResults" :key="m.id" @tap="jumpToMessage(m,i)">{{m.text||m.title||'图片或其他消息'}}</view><text v-if="!searchResults.length">没有找到相关内容</text></view><text class="search-cancel" @tap="searchModal=false">完成</text></view></view>
  <view v-if="toast" class="toast" @tap="toast=''">{{toast}}</view>
</view>
</template>
<script setup>
import{ref,nextTick,computed}from'vue'
import{onLoad,onShow}from'@dcloudio/uni-app'
const chatBg=ref('#ededed')
const name=ref('小鱼'),draft=ref(''),toast=ref(''),showTools=ref(false),showEmoji=ref(false),emojiExpanded=ref(false),emojiTab=ref('emoji'),recentEmojis=ref(['😂','❤️','🥰','👍','😊','😭','🎉','🙏','✨','😍','😉','🌹']),packetModal=ref(false),transferModal=ref(false),settingsVisible=ref(false),searchModal=ref(false),searchKeyword=ref(''),muted=ref(false),pinned=ref(false),packetType=ref('normal'),moneyAmount=ref(''),moneyGreeting=ref('恭喜发财，大吉大利') ,peerStatus=ref('对方在线'),lastMessageId=ref(''),myAvatar=ref('/static/avatar1.svg'),peerAvatar=ref('/static/avatar2.svg')
const searchResults=computed(()=>list.value.filter(m=>(m.text||m.title||'').toLowerCase().includes(searchKeyword.value.toLowerCase())))
const emojis=['😀','😃','😄','😁','😆','😅','🤣','😂','🙂','🙃','😉','😊','😇','🥰','😍','🤩','😘','😗','☺️','😚','😋','😛','😜','🤪','😝','🤑','🤗','🤭','🤫','🤔','🤐','🤨','😐','😑','😶','😏','😒','🙄','😬','🤥','😌','😔','😪','🤤','😴','😷','🤒','🤕','🤢','🤮','🥵','🥶','😵','🤯','🤠','🥳','😎','🤓','🧐','😕','😟','🙁','☹️','😮','😯','😲','😳','🥺','😦','😧','😨','😰','😥','😢','😭','😱','😖','😣','😞','😓','😩','😫','🥱','😤','😡','😠','🤬','😈','👿','💀','☠️','💩','🤡','👻','👽','🤖','🎃','😺','😸','😹','😻','😼','😽','🙀','😿','😾','❤️','🧡','💛','💚','💙','💜','🖤','🤍','💔','💕','💞','💓','💗','💖','💘','💝','💟','👍','👎','👌','✌️','🤞','🤟','🤘','🤙','👈','👉','👆','👇','☝️','✋','🤚','🖐️','🖖','👋','🤏','💪','🙏','👏','🙌','👐','🤲','🤝','💅','🎉','✨','🔥','🌹','🌸','☀️','🌈','🍀','🐱']
const list=ref([{id:1,text:'嗨，刚刚看到你的动态～',mine:false},{id:2,text:'你好呀，很高兴认识你',mine:true},{id:3,text:'你也喜欢旅行吗？',mine:false}])
function storageKey(){return'meet-chat-'+name.value}
function loadMessages(){try{const saved=uni.getStorageSync(storageKey());if(saved)list.value=saved}catch(e){}}
onLoad((q={})=>{if(q.name)name.value=decodeURIComponent(q.name);loadMessages();chatBg.value=uni.getStorageSync('meet-chat-bg-'+name.value)||'#ededed';muted.value=!!uni.getStorageSync('meet-chat-muted-'+name.value);pinned.value=!!uni.getStorageSync('meet-chat-pinned-'+name.value);scrollBottom()})
onShow(()=>loadMessages())
function persist(){uni.setStorageSync(storageKey(),list.value);try{const chats=uni.getStorageSync('meet-chat-list')||[];const item=chats.find(x=>x.name===name.value);const last=list.value[list.value.length-1];if(item&&last){item.last=last.text||last.title||(last.kind==='image'?'[图片]':last.kind==='redpacket'?'[红包]':last.kind==='transfer'?'[转账]':'[消息]');item.time='刚刚';item.unread=0;uni.setStorageSync('meet-chat-list',chats)}}catch(e){}scrollBottom()}
function scrollBottom(){nextTick(()=>{lastMessageId.value='';nextTick(()=>{lastMessageId.value='message-bottom'})})}
function send(){const value=draft.value.trim();if(!value)return;if(list.value.length>=500){toast.value='聊天记录太多，请清理后再发送';return}list.value.push({id:Date.now(),text:value,mine:true,time:Date.now()});draft.value='';showTools.value=false;showEmoji.value=false;persist()}
function choosePhoto(){uni.chooseImage({count:9,sizeType:['compressed'],success:r=>{r.tempFilePaths.forEach((src,i)=>list.value.push({id:Date.now()+i,text:src,mine:true,kind:'image'}));persist();showTools.value=false}})}
function takePhoto(){uni.chooseImage({count:1,sourceType:['camera'],sizeType:['compressed'],success:r=>{list.value.push({id:Date.now(),text:r.tempFilePaths[0],mine:true,kind:'image'});persist();showTools.value=false},fail:()=>{toast.value='无法打开相机，请检查系统权限'}})}
function previewImage(src){const urls=list.value.filter(x=>x.kind==='image').map(x=>x.text);uni.previewImage({current:src,urls})}
function sendLocation(){showTools.value=false;if(typeof uni.chooseLocation==='function'){uni.chooseLocation({success:r=>{list.value.push({id:Date.now(),text:'位置：'+r.name+' '+r.address,mine:true,kind:'location',latitude:r.latitude,longitude:r.longitude});persist()},fail:()=>{toast.value='未选择位置或定位权限未开启'}})}else toast.value='当前运行环境暂不支持地图选点'}
function chooseFile(){toast.value='文件选择需要按 H5、iOS、Android 分别接入系统文件选择器';showTools.value=false}
function openPacketForm(){moneyAmount.value='';moneyGreeting.value='恭喜发财，大吉大利';packetModal.value=true;showTools.value=false}
function openTransferForm(){moneyAmount.value='';moneyGreeting.value='';transferModal.value=true;showTools.value=false}
function validAmount(){const n=Number(moneyAmount.value);return Number.isFinite(n)&&n>0&&n<=20000&&/^\d+(\.\d{1,2})?$/.test(moneyAmount.value)}
function sendPacket(){if(!validAmount()){toast.value='请输入有效金额，最多两位小数';return}list.value.push({id:Date.now(),kind:'redpacket',mine:true,amount:Number(moneyAmount.value).toFixed(2),title:moneyGreeting.value.trim()||'恭喜发财，大吉大利',packetType:packetType.value,claimed:false});packetModal.value=false;persist()}
function sendTransfer(){if(!validAmount()){toast.value='请输入有效转账金额，最多两位小数';return}list.value.push({id:Date.now(),kind:'transfer',mine:true,amount:Number(moneyAmount.value).toFixed(2),note:moneyGreeting.value.trim(),claimed:false});transferModal.value=false;persist()}
function openPacket(m){if(m.mine){toast.value='这是你发出的红包，当前为演示状态';return}m.claimed=true;persist();toast.value='红包已领取（本机演示）'}
function sendFavorite(){list.value.push({id:Date.now(),text:'分享了一条收藏内容',mine:true});persist();showTools.value=false}
function insertEmoji(e){draft.value+=e;recentEmojis.value=[e,...recentEmojis.value.filter(x=>x!==e)].slice(0,24)}
function sendContact(){uni.showModal({title:'发送个人名片',content:'确认将自己的个人主页分享给对方？',success:r=>{if(r.confirm){list.value.push({id:Date.now(),text:'分享了个人主页',mine:true});persist()}}});showTools.value=false}
function chatMenu(){settingsVisible.value=true;showEmoji.value=false;showTools.value=false}
function searchHistory(){settingsVisible.value=false;searchKeyword.value='';searchModal.value=true}
function jumpToMessage(m){searchModal.value=false;const i=list.value.indexOf(m);lastMessageId.value='msg-'+i}
function toggleMute(){muted.value=!muted.value;uni.setStorageSync('meet-chat-muted-'+name.value,muted.value);settingsVisible.value=false;toast.value=muted.value?'已开启消息免打扰':'已恢复消息提醒'}
function toggleTop(){pinned.value=!pinned.value;uni.setStorageSync('meet-chat-pinned-'+name.value,pinned.value);settingsVisible.value=false;toast.value=pinned.value?'已置顶聊天':'已取消置顶'}
function changeBackground(){settingsVisible.value=false;uni.showActionSheet({itemList:['默认背景','浅灰背景','暖白背景'],success:r=>{const colors=['#ededed','#e9eef2','#f5f1eb'];uni.setStorageSync('meet-chat-bg-'+name.value,colors[r.tapIndex]);chatBg.value=colors[r.tapIndex];toast.value='聊天背景已更新'}})}
function clearHistory(){settingsVisible.value=false;uni.showModal({title:'清空聊天记录',content:'确定清空本机保存的聊天记录吗？',success:x=>{if(x.confirm){list.value=[];uni.removeStorageSync(storageKey())}}})}
function messageActions(m,i){const actions=['复制','收藏','转发','删除'];if(m.kind==='image')actions[0]='查看图片';uni.showActionSheet({itemList:actions,success:r=>{if(r.tapIndex===0){if(m.kind==='image')previewImage(m.text);else if(typeof uni.setClipboardData==='function')uni.setClipboardData({data:m.text||m.title||''});else toast.value='当前环境不支持复制'}if(r.tapIndex===1){const saved=uni.getStorageSync('meet-favorites')||[];saved.unshift({...m,savedAt:Date.now(),chatName:name.value});uni.setStorageSync('meet-favorites',saved);toast.value='已收藏'}if(r.tapIndex===2){uni.showActionSheet({itemList:['转发给其他好友'],success:()=>{toast.value='转发对象选择需要接入联系人列表'}})}if(r.tapIndex===3){list.value.splice(i,1);persist()}}})}
function back(){uni.navigateBack()}
</script>
<style scoped lang="scss">
.chat-page{height:100vh;height:100dvh;display:flex;flex-direction:column;background:#ededed;color:#191919;overflow:hidden;position:relative}.topbar{height:54px;padding:0 14px;display:flex;align-items:center;justify-content:space-between;background:#f7f7f7;border-bottom:1px solid #e8e8e8;flex-shrink:0}.back{font-size:31px;width:38px;color:#222;line-height:1}.top-title{flex:1;text-align:center}.title{display:block;font-size:16px;font-weight:650}.online{display:block;font-size:9px;color:#999;margin-top:3px}.more{width:38px;text-align:right;font-size:25px;letter-spacing:1px}.message-scroll{flex:1;min-height:0;padding:12px 13px;box-sizing:border-box}.history-hint{text-align:center;color:#b2b2b2;font-size:10px;margin:9px 0 16px}.time-label{text-align:center;font-size:10px;color:#aaa;margin:18px 0}.message-row{display:flex;align-items:flex-start;gap:9px;margin:16px 0}.message-row.mine{flex-direction:row-reverse}.avatar{width:39px;height:39px;border-radius:5px;background:#d9d9d9;flex-shrink:0}.message-content{max-width:72%;display:flex;flex-direction:column}.bubble{padding:10px 12px;border-radius:4px 5px 5px 5px;background:#fff;font-size:14px;line-height:1.55;word-break:break-word;position:relative}.mine .bubble{background:#95ec69;border-radius:5px 4px 5px 5px}.message-state{font-size:9px;color:#aaa;margin-top:4px}.mine .message-state{text-align:right}.image-bubble image{display:block;max-width:190px;max-height:230px;border-radius:4px}.voice-bubble{min-width:92px;padding:10px 12px;background:#fff;border-radius:5px;display:flex;justify-content:space-between;font-size:14px}.mine .voice-bubble{background:#95ec69}.bottom-anchor{height:2px}.tool-panel{background:#f6f6f6;border-top:1px solid #e5e5e5;padding:18px 14px 22px;display:grid;grid-template-columns:repeat(4,1fr);gap:20px 8px}.tool-item{display:flex;flex-direction:column;align-items:center;gap:7px;font-size:10px;color:#666}.tool-icon{width:48px;height:48px;border-radius:12px;background:#fff;display:grid;place-items:center;font-size:24px;color:#555}.tool-icon.redpacket{color:#e96b55}.tool-icon.file{color:#7b9bb9}.tool-icon.transfer{color:#6ca7a0}.tool-icon.favorite{color:#d5a65f}.emoji-panel{background:#ededed;padding:0 8px 8px;display:flex;flex-direction:column;flex-shrink:0}
.emoji-grabber{height:17px;display:flex;align-items:center;justify-content:center}.emoji-grabber view{width:34px;height:4px;border-radius:5px;background:#c7c7c7}
.emoji-tabs{height:43px;display:flex;align-items:center;border-bottom:1px solid #ddd;gap:24px;padding:0 8px;font-size:21px;color:#777}.emoji-tabs text.selected{color:#07c160}.emoji-manage{margin-left:auto}
.emoji-grid{height:190px}.emoji-grid.expanded{height:340px}.emoji-cells{display:grid;grid-template-columns:repeat(8,1fr);text-align:center;row-gap:13px;padding:10px 0;font-size:25px}.emoji-cells text{height:32px}
.sticker-placeholder{height:100%;display:flex;flex-direction:column;align-items:center;justify-content:center;color:#999;gap:10px}.sticker-placeholder>text{font-size:46px}.sticker-placeholder small{font-size:11px}.sticker-placeholder button{font-size:11px;line-height:32px;height:32px;background:#fff;border:0;border-radius:5px;color:#555}
.emoji-footer{height:44px;display:flex;align-items:center;justify-content:flex-end;gap:12px}.emoji-footer text{font-size:11px;color:#777}.emoji-footer button{margin:0;background:#07c160;color:#fff;border:0;font-size:12px;line-height:32px;height:32px;padding:0 23px}.emoji-footer button[disabled]{background:#b5d9c4}.composer{min-height:56px;background:#f7f7f7;border-top:1px solid #e4e4e4;display:flex;align-items:center;gap:9px;padding:7px 10px calc(7px + env(safe-area-inset-bottom));box-sizing:content-box;flex-shrink:0}.emoji-button,.plus-button{font-size:25px;width:30px;text-align:center;color:#333;flex-shrink:0}.message-input{height:39px;min-width:0;flex:1;background:#fff;border-radius:5px;padding:0 10px;font-size:14px}.send-button{background:#07c160;color:#fff;border-radius:5px;padding:9px 12px;font-size:13px;line-height:20px;flex-shrink:0;margin-bottom:1px}.redpacket-message,.transfer-message{width:218px;border-radius:5px;overflow:hidden;background:#f5a64a;color:#fff}.packet-top,.transfer-top{height:68px;display:flex;align-items:center;gap:12px;padding:0 13px}.packet-symbol{font-size:30px}.packet-top b,.packet-top small,.transfer-top b,.transfer-top small{display:block}.packet-top b,.transfer-top b{font-size:14px;font-weight:600}.packet-top small,.transfer-top small{font-size:10px;opacity:.85;margin-top:5px}.packet-bottom,.transfer-bottom{height:27px;background:#fff9f0;color:#b18a60;font-size:9px;padding:7px 12px;box-sizing:border-box}.transfer-message{background:#e9b85d}.transfer-top>text{font-size:30px}.modal-mask{position:fixed;z-index:40;inset:0;background:#0007;display:flex;align-items:center;justify-content:center;padding:20px;box-sizing:border-box}.money-modal{width:100%;max-width:360px;background:#fff;border-radius:14px;padding:20px;box-sizing:border-box}.packet-modal{background:#fff8ee}.money-head{text-align:center;font-size:17px;font-weight:700;color:#4b4037;margin-bottom:18px}.packet-type{display:flex;gap:10px;margin-bottom:18px}.packet-type text{flex:1;text-align:center;padding:9px 4px;border:1px solid #e8ddd0;border-radius:8px;font-size:11px;color:#8b8177}.packet-type .active{border-color:#e8a34b;background:#fff0d9;color:#c47c2e}.money-label{font-size:11px;color:#8b8177}.money-input{display:flex;align-items:center;border-bottom:1px solid #eee1d2;padding:5px 0 12px;margin-bottom:15px}.money-input>text{font-size:26px;font-weight:700;color:#3d352f;margin-right:8px}.money-input input{font-size:29px;font-weight:700;height:43px;flex:1;min-width:0;color:#3d352f}.money-note{background:#f7f5f2;border-radius:8px;padding:0 10px;margin:10px 0}.money-note input{height:42px;font-size:12px}.money-explain{font-size:10px;color:#b1a69b;line-height:1.6;margin:12px 0}.money-submit{height:44px;line-height:44px;border:0;border-radius:7px;background:#e9a24a;color:white;font-size:14px;margin-top:9px}.transfer-submit{background:#08a85a}.money-cancel{display:block;text-align:center;color:#999;font-size:12px;padding-top:14px}.toast{position:fixed;z-index:50;bottom:90px;left:10%;width:80%;text-align:center;background:#333e;color:#fff;padding:11px;border-radius:7px;font-size:12px}

/* Refined WeChat-inspired conversation details */
.topbar{height:52px;min-height:52px;background:#ededed;border-bottom:1px solid #dedede;padding:0 13px;padding-top:env(safe-area-inset-top);box-sizing:content-box}
.back{font-size:29px;width:34px}
.title{font-size:17px;font-weight:600}
.online{display:none}
.more{font-size:25px}
.message-scroll{padding:10px 12px}
.history-hint{font-size:11px;margin:14px 0 22px;color:#b4b4b4}
.time-label{font-size:11px;color:#b2b2b2;margin:20px 0}
.message-row{gap:9px;margin:18px 0;align-items:flex-start}
.avatar{width:40px;height:40px;border-radius:4px;flex:0 0 40px}
.message-content{max-width:calc(100% - 55px);min-width:0}
.bubble{font-size:16px;line-height:1.5;padding:9px 12px;border-radius:5px;background:#fff;position:relative;white-space:pre-wrap;overflow-wrap:anywhere;box-shadow:0 1px 1px #00000008}
.mine .bubble{background:#95ec69;border-radius:5px}
.message-state{display:none}
.tool-panel{padding:18px 17px calc(25px + env(safe-area-inset-bottom));gap:22px 8px;background:#ededed;flex-shrink:0}
.tool-item{font-size:11px;color:#555;gap:8px}
.tool-icon{width:52px;height:52px;border-radius:10px;font-size:25px;background:#fff}
.composer{min-height:54px;background:#f7f7f7;padding:7px 10px calc(7px + env(safe-area-inset-bottom));gap:8px;align-items:flex-end;box-sizing:border-box}
.message-input{min-height:38px;max-height:112px;line-height:22px;box-sizing:border-box;border-radius:5px;background:#fff;font-size:16px;padding:8px 10px;flex:1;min-width:0;overflow-y:auto}
.emoji-button,.plus-button{font-size:27px;width:31px;height:38px;line-height:38px;flex-shrink:0}
.sheet-mask,.search-mask{position:fixed;z-index:45;inset:0;background:#0005;display:flex;align-items:flex-end;justify-content:center}
.settings-sheet{width:100%;background:#f7f7f7;border-radius:15px 15px 0 0;padding:9px 16px calc(18px + env(safe-area-inset-bottom));box-sizing:border-box}
.sheet-grabber{width:34px;height:4px;border-radius:4px;background:#d1d1d1;margin:0 auto 13px}
.sheet-title{text-align:center;font-size:16px;font-weight:600;margin-bottom:20px}
.settings-list{background:#f7f7f7;border-radius:8px;overflow:hidden}
.settings-item{height:53px;background:#fff;display:flex;align-items:center;justify-content:space-between;padding:0 15px;border-bottom:1px solid #eee;font-size:13px;color:#222}
.settings-item.arrow>text:last-child{font-size:22px;color:#bbb}
.settings-item switch{transform:scale(.82);transform-origin:right center}
.settings-gap{height:10px}
.settings-sheet{padding-left:0;padding-right:0}
.sheet-title{margin-bottom:15px}
.search-mask{align-items:center;padding:22px;box-sizing:border-box}
.search-box{width:100%;max-width:400px;max-height:75vh;background:#fff;border-radius:12px;padding:19px;box-sizing:border-box;display:flex;flex-direction:column}
.search-title{text-align:center;font-size:16px;font-weight:600;margin-bottom:17px}
.search-input{display:flex;align-items:center;background:#f2f2f2;border-radius:7px;padding:0 10px;height:40px}
.search-input input{flex:1;min-width:0;font-size:13px}
.search-input text{font-size:10px;color:#999}
.search-results{overflow:auto;max-height:42vh}
.search-results view{padding:12px 2px;border-bottom:1px solid #eee;font-size:12px;color:#555}
.search-results>text{display:block;text-align:center;color:#aaa;padding:25px 0;font-size:12px}
.search-cancel{display:block;text-align:center;padding-top:17px;color:#576b95;font-size:13px}
</style>