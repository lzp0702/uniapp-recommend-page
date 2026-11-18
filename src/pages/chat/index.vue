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
  <view v-if="packetModal" class="modal-mask money-mask" @tap="packetModal=false"><view class="packet-sheet" @tap.stop><view class="money-sheet-bar"><text @tap="packetModal=false">取消</text><text>发红包</text><text></text></view><view class="packet-recipient">发给 <text>{{name}}</text></view><view class="packet-amount-label">金额</view><view class="packet-amount"><text>¥</text><input v-model="moneyAmount" type="digit" placeholder="0.00"/></view><view class="packet-greeting"><input v-model="moneyGreeting" maxlength="24" placeholder="恭喜发财，大吉大利"/></view><view class="packet-demo-note">仅为界面演示，不会扣款或产生真实资金交易</view><button class="packet-confirm" @tap="sendPacket">塞钱进红包</button></view></view>
  <view v-if="transferModal" class="modal-mask money-mask" @tap="transferModal=false"><view class="transfer-sheet" @tap.stop><view class="money-sheet-bar transfer-bar"><text @tap="transferModal=false">取消</text><text>转账</text><text></text></view><view class="transfer-recipient"><view class="transfer-avatar">{{name.slice(0,1)}}</view><text>转账给 {{name}}</text></view><view class="transfer-amount-label">转账金额</view><view class="transfer-amount"><text>¥</text><input v-model="moneyAmount" type="digit" placeholder="0.00"/></view><view class="transfer-note"><input v-model="moneyGreeting" maxlength="24" placeholder="添加转账说明（选填）"/></view><view class="transfer-demo-note">演示转账仅生成聊天记录，不涉及真实支付</view><button class="transfer-confirm" @tap="sendTransfer">转账</button></view></view>
  <view v-if="replyingTo" class="reply-preview"><view><text>回复 {{replyingTo.mine?'自己':name}}</text><text class="reply-snippet">{{replyingTo.text||'图片/其他消息'}}</text></view><text @tap="replyingTo=null">×</text></view>
  <view v-if="selectedMessage" class="message-context-mask" @tap="closeMessageMenu"><view class="context-card" @tap.stop><view class="reaction-strip"><text v-for="e in quickReactions" :key="e" @tap="reactTo(selectedMessage,e)">{{e}}</text></view><view class="context-actions"><view @tap="replyMessage">回复 <text>↩</text></view><view @tap="copyMessage">复制 <text>▢</text></view><view @tap="forwardMessage">转发 <text>➦</text></view><view @tap="favoriteMessage">收藏 <text>☆</text></view><view class="danger" @tap="deleteMessage">删除 <text>⌫</text></view></view></view></view>
  <view class="composer" :class="{'panel-open':showEmoji||showTools}">
    <view class="composer-face" @tap="showEmoji=!showEmoji;showTools=false">☺</view>
    <textarea v-model="draft" class="message-input" auto-height maxlength="-1" :show-confirm-bar="false" cursor-spacing="8" placeholder="发送消息" @confirm="send"/>
    <view v-if="!draft.trim()" class="plus-button" @tap="showTools=!showTools;showEmoji=false">＋</view>
    <view v-else class="send-button" @tap="send">发送</view>
  </view>
  <view v-if="showEmoji" class="emoji-panel">
    <view class="emoji-grabber" @tap="emojiExpanded=!emojiExpanded"><view></view></view>
    <view class="emoji-tabs"><text :class="{selected:emojiTab==='recent'}" @tap="emojiTab='recent'">◷</text><text :class="{selected:emojiTab==='emoji'}" @tap="emojiTab='emoji'">☺</text><text :class="{selected:emojiTab==='stickers'}" @tap="emojiTab='stickers'">▧</text><text class="emoji-manage" @tap="toast='表情管理入口：可在正式版本接入表情商店'">＋</text></view>
    <scroll-view class="emoji-grid" scroll-y :class="{expanded:emojiExpanded}">
      <view v-if="emojiTab==='recent'||emojiTab==='emoji'" class="emoji-cells"><text v-for="e in (emojiTab==='recent'?recentEmojis:emojis)" :key="e" @tap="insertEmoji(e)">{{e}}</text></view>
      <view v-else class="sticker-placeholder"><text>☺</text><small>暂无添加的表情包</small><button @tap="toast='表情商店需要接入表情资源'">添加表情</button></view>
    </scroll-view>
    <view class="emoji-footer"><text @tap="draft=draft.slice(0,-2)">删除</text><button :disabled="!draft.trim()" @tap="send">发送</button></view>
  </view>
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
const chatBg=ref('#ededed'),selectedMessage=ref(null),replyingTo=ref(null),quickReactions=['❤️','👍','😂','😮','😭','🙏','🔥']
const name=ref('小鱼'),draft=ref(''),toast=ref(''),showTools=ref(false),showEmoji=ref(false),emojiExpanded=ref(false),emojiTab=ref('emoji'),recentEmojis=ref(['😂','❤️','🥰','👍','😊','😭','🎉','🙏','✨','😍','😉','🌹']),packetModal=ref(false),transferModal=ref(false),settingsVisible=ref(false),searchModal=ref(false),searchKeyword=ref(''),muted=ref(false),pinned=ref(false),moneyAmount=ref(''),moneyGreeting=ref('恭喜发财，大吉大利') ,peerStatus=ref('对方在线'),lastMessageId=ref(''),myAvatar=ref('/static/avatar1.svg'),peerAvatar=ref('/static/avatar2.svg')
const searchResults=computed(()=>list.value.filter(m=>(m.text||m.title||'').toLowerCase().includes(searchKeyword.value.toLowerCase())))
const emojis=['😀','😃','😄','😁','😆','😅','🤣','😂','🙂','🙃','😉','😊','😇','🥰','😍','🤩','😘','😗','☺️','😚','😋','😛','😜','🤪','😝','🤑','🤗','🤭','🤫','🤔','🤐','🤨','😐','😑','😶','😏','😒','🙄','😬','🤥','😌','😔','😪','🤤','😴','😷','🤒','🤕','🤢','🤮','🥵','🥶','😵','🤯','🤠','🥳','😎','🤓','🧐','😕','😟','🙁','☹️','😮','😯','😲','😳','🥺','😦','😧','😨','😰','😥','😢','😭','😱','😖','😣','😞','😓','😩','😫','🥱','😤','😡','😠','🤬','😈','👿','💀','☠️','💩','🤡','👻','👽','🤖','🎃','😺','😸','😹','😻','😼','😽','🙀','😿','😾','❤️','🧡','💛','💚','💙','💜','🖤','🤍','💔','💕','💞','💓','💗','💖','💘','💝','💟','👍','👎','👌','✌️','🤞','🤟','🤘','🤙','👈','👉','👆','👇','☝️','✋','🤚','🖐️','🖖','👋','🤏','💪','🙏','👏','🙌','👐','🤲','🤝','💅','🎉','✨','🔥','🌹','🌸','☀️','🌈','🍀','🐱']
const list=ref([{id:1,text:'嗨，刚刚看到你的动态～',mine:false},{id:2,text:'你好呀，很高兴认识你',mine:true},{id:3,text:'你也喜欢旅行吗？',mine:false}])
function storageKey(){return'meet-chat-'+name.value}
function loadMessages(){try{const saved=uni.getStorageSync(storageKey());if(saved)list.value=saved}catch(e){}}
onLoad((q={})=>{if(q.name)name.value=decodeURIComponent(q.name);loadMessages();chatBg.value=uni.getStorageSync('meet-chat-bg-'+name.value)||'#ededed';muted.value=!!uni.getStorageSync('meet-chat-muted-'+name.value);pinned.value=!!uni.getStorageSync('meet-chat-pinned-'+name.value);scrollBottom()})
onShow(()=>loadMessages())
function persist(){uni.setStorageSync(storageKey(),list.value);try{const chats=uni.getStorageSync('meet-chat-list')||[];const item=chats.find(x=>x.name===name.value);const last=list.value[list.value.length-1];if(item&&last){item.last=last.text||last.title||(last.kind==='image'?'[图片]':last.kind==='redpacket'?'[红包]':last.kind==='transfer'?'[转账]':'[消息]');item.time='刚刚';item.unread=0;uni.setStorageSync('meet-chat-list',chats)}}catch(e){}scrollBottom()}
function scrollBottom(){nextTick(()=>{lastMessageId.value='';nextTick(()=>{lastMessageId.value='message-bottom'})})}
function send(){const value=draft.value.trim();if(!value)return;if(list.value.length>=500){toast.value='聊天记录太多，请清理后再发送';return}list.value.push({id:Date.now(),text:value,mine:true,time:Date.now(),reply:replyingTo.value?{text:replyingTo.value.text,mine:replyingTo.value.mine}:null});draft.value='';replyingTo.value=null;showTools.value=false;showEmoji.value=false;persist()}
function choosePhoto(){uni.chooseImage({count:9,sizeType:['compressed'],success:r=>{r.tempFilePaths.forEach((src,i)=>list.value.push({id:Date.now()+i,text:src,mine:true,kind:'image'}));persist();showTools.value=false}})}
function takePhoto(){uni.chooseImage({count:1,sourceType:['camera'],sizeType:['compressed'],success:r=>{list.value.push({id:Date.now(),text:r.tempFilePaths[0],mine:true,kind:'image'});persist();showTools.value=false},fail:()=>{toast.value='无法打开相机，请检查系统权限'}})}
function previewImage(src){const urls=list.value.filter(x=>x.kind==='image').map(x=>x.text);uni.previewImage({current:src,urls})}
function sendLocation(){showTools.value=false;if(typeof uni.chooseLocation==='function'){uni.chooseLocation({success:r=>{list.value.push({id:Date.now(),text:'位置：'+r.name+' '+r.address,mine:true,kind:'location',latitude:r.latitude,longitude:r.longitude});persist()},fail:()=>{toast.value='未选择位置或定位权限未开启'}})}else toast.value='当前运行环境暂不支持地图选点'}
function chooseFile(){toast.value='文件选择需要按 H5、iOS、Android 分别接入系统文件选择器';showTools.value=false}
function openPacketForm(){moneyAmount.value='';moneyGreeting.value='恭喜发财，大吉大利';packetModal.value=true;showTools.value=false}
function openTransferForm(){moneyAmount.value='';moneyGreeting.value='';transferModal.value=true;showTools.value=false}
function validAmount(){const n=Number(moneyAmount.value);return Number.isFinite(n)&&n>0&&n<=20000&&/^\d+(\.\d{1,2})?$/.test(moneyAmount.value)}
function sendPacket(){if(!validAmount()){toast.value='请输入有效金额，最多两位小数';return}list.value.push({id:Date.now(),kind:'redpacket',mine:true,amount:Number(moneyAmount.value).toFixed(2),title:moneyGreeting.value.trim()||'恭喜发财，大吉大利',claimed:false});packetModal.value=false;persist()}
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
function messageActions(m,i){selectedMessage.value={...m,_index:i};showEmoji.value=false;showTools.value=false}
function closeMessageMenu(){selectedMessage.value=null}
function reactTo(m,e){const target=list.value.find(x=>x.id===m.id);if(target){target.reaction=e;persist()}selectedMessage.value=null}
function replyMessage(){if(selectedMessage.value){replyingTo.value=selectedMessage.value;draft.value='';selectedMessage.value=null}}
function copyMessage(){const m=selectedMessage.value;if(m&&typeof uni.setClipboardData==='function')uni.setClipboardData({data:m.text||m.title||''});selectedMessage.value=null}
function forwardMessage(){selectedMessage.value=null;toast.value='转发对象选择需要接入联系人列表'}
function favoriteMessage(){const m=selectedMessage.value;if(m){const saved=uni.getStorageSync('meet-favorites')||[];saved.unshift({...m,savedAt:Date.now(),chatName:name.value});uni.setStorageSync('meet-favorites',saved);toast.value='已收藏'}selectedMessage.value=null}
function deleteMessage(){const m=selectedMessage.value;if(m){list.value.splice(m._index,1);persist()}selectedMessage.value=null}
function back(){uni.navigateBack()}
</script>
<style scoped lang="scss">
.chat-page{height:100vh;height:100dvh;display:flex;flex-direction:column;overflow:hidden;color:#202124;background:#ededed;position:relative}
.topbar{height:48px;min-height:48px;padding:env(safe-area-inset-top) 14px 0;background:#f7f7f7;border-bottom:1px solid #e7e7e7;display:flex;align-items:center;justify-content:space-between;flex-shrink:0;box-sizing:content-box}
.back{width:34px;font-size:30px;line-height:1;color:#222}.top-title{flex:1;text-align:center}.title{display:block;font-size:16px;font-weight:600}.online{display:block;font-size:10px;color:#999;margin-top:2px}.more{width:34px;text-align:right;font-size:24px}
.message-scroll{flex:1;min-height:0;padding:12px 12px 16px;box-sizing:border-box}.history-hint{text-align:center;color:#a5a5a5;font-size:11px;margin:10px 0 22px}.time-label{text-align:center;color:#a0a0a0;font-size:11px;margin:18px 0}
.message-row{display:flex;align-items:flex-start;gap:9px;margin:12px 0}.message-row.mine{flex-direction:row-reverse}.avatar{width:40px;height:40px;flex:0 0 40px;border-radius:7px;background:#ddd}.message-content{max-width:78%;min-width:0;display:flex;flex-direction:column}.bubble{max-width:100%;padding:9px 12px;border-radius:5px 16px 16px 16px;background:#fff;color:#202124;font-size:15px;line-height:1.48;white-space:pre-wrap;overflow-wrap:anywhere;box-shadow:0 1px 1px #00000008}.mine .bubble{background:#95ec69;border-radius:16px 5px 16px 16px}
.message-state{font-size:10px;color:#999;margin:4px 3px 0}.mine .message-state{text-align:right}.image-bubble image{display:block;max-width:210px;max-height:260px;border-radius:10px}.reaction-chip{align-self:flex-start;margin-top:4px;background:#fff;border:1px solid #e9e9e9;border-radius:12px;padding:2px 7px;font-size:12px;color:#666}
.composer{min-height:54px;background:#f7f7f7;border-top:1px solid #e6e6e6;display:flex;align-items:flex-end;gap:8px;padding:7px 10px calc(7px + env(safe-area-inset-bottom));box-sizing:border-box;flex-shrink:0}.composer-face,.plus-button{height:38px;width:32px;flex:0 0 32px;display:grid;place-items:center;font-size:27px;color:#303030}.message-input{flex:1;min-width:0;min-height:38px;max-height:112px;line-height:22px;box-sizing:border-box;border-radius:7px;background:#fff;font-size:16px;padding:8px 10px;overflow-y:auto}.send-button{background:#07c160;color:#fff;border-radius:7px;padding:9px 13px;font-size:13px;line-height:20px;flex-shrink:0}
.tool-panel{background:#f2f2f2;border-top:1px solid #e5e5e5;padding:18px 16px calc(20px + env(safe-area-inset-bottom));display:grid;grid-template-columns:repeat(4,1fr);gap:22px 8px;flex-shrink:0}.tool-item{display:flex;flex-direction:column;align-items:center;gap:8px;font-size:11px;color:#555}.tool-icon{width:52px;height:52px;border-radius:13px;background:#fff;display:grid;place-items:center;font-size:24px;color:#555}
.emoji-panel{background:#ededed;padding:0 8px 8px;display:flex;flex-direction:column;flex-shrink:0}.emoji-grabber{height:17px;display:flex;align-items:center;justify-content:center}.emoji-grabber view{width:34px;height:4px;border-radius:5px;background:#c7c7c7}.emoji-tabs{height:43px;display:flex;align-items:center;border-bottom:1px solid #ddd;gap:24px;padding:0 8px;font-size:21px;color:#777}.emoji-tabs text.selected{color:#07c160}.emoji-manage{margin-left:auto}.emoji-grid{height:190px}.emoji-grid.expanded{height:340px}.emoji-cells{display:grid;grid-template-columns:repeat(8,1fr);text-align:center;row-gap:13px;padding:10px 0;font-size:25px}.emoji-cells text{height:32px}.sticker-placeholder{height:100%;display:flex;flex-direction:column;align-items:center;justify-content:center;color:#999;gap:10px}.sticker-placeholder>text{font-size:46px}.sticker-placeholder small{font-size:11px}.sticker-placeholder button{font-size:11px;line-height:32px;height:32px;background:#fff;border:0;border-radius:5px;color:#555}.emoji-footer{height:44px;display:flex;align-items:center;justify-content:flex-end;gap:12px}.emoji-footer text{font-size:11px;color:#777}.emoji-footer button{margin:0;background:#07c160;color:#fff;border:0;font-size:12px;line-height:32px;height:32px;padding:0 23px}
.reply-preview{display:flex;align-items:center;justify-content:space-between;background:#f6f6f6;border-top:1px solid #e8e8e8;padding:7px 14px;color:#777;font-size:11px}.reply-preview>view{display:flex;flex-direction:column;gap:3px;border-left:3px solid #07c160;padding-left:8px}.reply-snippet{max-width:75vw;white-space:nowrap;overflow:hidden;text-overflow:ellipsis}.reply-preview>text{font-size:22px;padding:0 5px}
.message-context-mask{position:fixed;z-index:35;inset:0;background:#0005;display:flex;align-items:center;justify-content:center;padding:24px;box-sizing:border-box}.context-card{width:100%;max-width:330px;background:#f8f8f8;border-radius:18px;padding:9px;box-shadow:0 12px 40px #0003}.reaction-strip{display:flex;justify-content:space-around;align-items:center;background:#fff;border-radius:14px;padding:10px 3px;margin-bottom:8px}.reaction-strip text{font-size:25px;padding:0 2px}.context-actions{background:#fff;border-radius:13px;overflow:hidden}.context-actions view{height:48px;padding:0 15px;display:flex;align-items:center;justify-content:space-between;border-bottom:1px solid #eee;font-size:14px;color:#242424}.context-actions view:last-child{border:0}.context-actions view text{font-size:18px;color:#888}.context-actions .danger{color:#e45454}
.redpacket-message,.transfer-message{width:226px;border-radius:9px;overflow:hidden;color:#fff;box-shadow:0 1px 2px #00000012}
.redpacket-message{background:#fa9d3b}.packet-top,.transfer-top{height:72px;display:flex;align-items:center;gap:12px;padding:0 14px}.packet-symbol,.transfer-top>text{font-size:30px}.packet-top b,.packet-top small,.transfer-top b,.transfer-top small{display:block}.packet-top b,.transfer-top b{font-size:14px}.packet-top small,.transfer-top small{font-size:10px;opacity:.88;margin-top:5px}.packet-bottom,.transfer-bottom{height:28px;background:#fff9f0;color:#a88963;font-size:10px;padding:7px 12px;box-sizing:border-box}.transfer-message{background:#e8b85f}
.money-mask{padding:0 18px;align-items:center;background:#0007}.packet-sheet,.transfer-sheet{width:100%;max-width:390px;border-radius:14px;overflow:hidden;box-sizing:border-box;box-shadow:0 16px 48px #0003}
.packet-sheet{background:#fff8f0;padding:0 22px 25px}.transfer-sheet{background:#fff;padding:0 22px 25px}
.money-sheet-bar{height:54px;display:grid;grid-template-columns:1fr 1fr 1fr;align-items:center;border-bottom:1px solid #f0e5d8;color:#4b4037;font-size:15px;font-weight:600;text-align:center}.money-sheet-bar text:first-child{text-align:left;font-size:13px;font-weight:400;color:#8e8174}.money-sheet-bar text:last-child{min-height:1px}
.packet-recipient{text-align:center;color:#8c8177;font-size:13px;padding:22px 0 27px}.packet-recipient text{color:#4b4037;font-weight:600}.packet-amount-label,.transfer-amount-label{font-size:13px;color:#766d64;margin-bottom:7px}
.packet-amount,.transfer-amount{height:72px;display:flex;align-items:center;border-bottom:1px solid #e9dfd5;margin-bottom:20px}.packet-amount>text,.transfer-amount>text{font-size:34px;font-weight:600;margin-right:10px;color:#302b27}.packet-amount input,.transfer-amount input{height:62px;flex:1;min-width:0;font-size:42px;font-weight:600;color:#302b27}
.packet-greeting,.transfer-note{height:48px;display:flex;align-items:center;background:#fff;border:1px solid #eee5dc;border-radius:7px;padding:0 12px;box-sizing:border-box}.packet-greeting input,.transfer-note input{font-size:13px;width:100%;height:46px;color:#333}
.packet-demo-note,.transfer-demo-note{text-align:center;color:#aaa;font-size:10px;margin:13px 0 17px;line-height:1.5}
.packet-confirm,.transfer-confirm{height:46px;line-height:46px;border:0;border-radius:7px;font-size:15px;font-weight:600;margin:0;width:100%}.packet-confirm{background:#eaa04a;color:#fff}.transfer-confirm{background:#07c160;color:#fff}
.transfer-bar{border-color:#eee;color:#222}.transfer-bar text:first-child{color:#777}.transfer-recipient{display:flex;align-items:center;justify-content:center;gap:10px;padding:25px 0 29px;font-size:14px;color:#333}.transfer-avatar{width:34px;height:34px;border-radius:7px;background:#d9e9df;color:#159451;display:flex;align-items:center;justify-content:center;font-size:16px;font-weight:600}
.transfer-amount{margin-bottom:19px;border-color:#e9e9e9}.transfer-amount-label{color:#777}.transfer-note{background:#f7f7f7;border-color:#f1f1f1}
.sheet-mask,.search-mask{position:fixed;z-index:45;inset:0;background:#0005;display:flex;align-items:flex-end;justify-content:center}.settings-sheet{width:100%;background:#f7f7f7;border-radius:16px 16px 0 0;padding:9px 0 calc(18px + env(safe-area-inset-bottom));box-sizing:border-box}.sheet-grabber{width:34px;height:4px;border-radius:4px;background:#d1d1d1;margin:0 auto 13px}.sheet-title{text-align:center;font-size:16px;font-weight:600;margin-bottom:15px}.settings-list{background:#f7f7f7;overflow:hidden}.settings-item{height:53px;background:#fff;display:flex;align-items:center;justify-content:space-between;padding:0 16px;border-bottom:1px solid #eee;font-size:13px;color:#222}.settings-item.arrow>text:last-child{font-size:22px;color:#bbb}.settings-item switch{transform:scale(.82);transform-origin:right center}.settings-gap{height:10px}
.search-mask{align-items:center;padding:22px;box-sizing:border-box}.search-box{width:100%;max-width:400px;max-height:75vh;background:#fff;border-radius:12px;padding:19px;box-sizing:border-box;display:flex;flex-direction:column}.search-title{text-align:center;font-size:16px;font-weight:600;margin-bottom:17px}.search-input{display:flex;align-items:center;background:#f2f2f2;border-radius:7px;padding:0 10px;height:40px}.search-input input{flex:1;min-width:0;font-size:13px}.search-input text{font-size:10px;color:#999}.search-results{overflow:auto;max-height:42vh}.search-results view{padding:12px 2px;border-bottom:1px solid #eee;font-size:12px;color:#555}.search-results>text{display:block;text-align:center;color:#aaa;padding:25px 0;font-size:12px}.search-cancel{display:block;text-align:center;padding-top:17px;color:#576b95;font-size:13px}.toast{position:fixed;z-index:50;bottom:90px;left:10%;width:80%;text-align:center;background:#333e;color:#fff;padding:11px;border-radius:7px;font-size:12px}
</style>