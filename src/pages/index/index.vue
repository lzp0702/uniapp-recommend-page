<template>
<view class="screen" :style="{'--safe-top':statusHeight+'px'}">
  <view class="ambient"></view>
  <view class="safe-top" :style="{height:statusHeight+'px'}"></view>
  <view class="topbar">
    <text class="heading">{{tabTitles[activeTab]}}</text>
    <view class="actions">
      <view class="action-box" @tap="openSettings"><text class="ticket">✦</text></view>
      <view class="action-box" @tap="showNotice"><text class="bell">♧</text></view>
      <text class="search" @tap="showSearch">⌕</text>
    </view>
  </view>

  <view v-if="activeTab===0" class="feed-wrap">
    <scroll-view class="feed" scroll-y :show-scrollbar="false" :refresher-enabled="true" :refresher-triggered="refreshing" lower-threshold="120" @refresherrefresh="refreshList" @scrolltolower="loadMore" @scroll="onScroll">
      <view class="banner"><view><text class="banner-kicker">TODAY'S CONNECTION</text><text class="banner-title">遇见同频的你</text><text class="banner-sub">每一次滑动，都可能遇见心动</text></view><text class="banner-orb">♡</text></view>
      <view class="filter-row"><text v-for="(f,i) in filters" :key="f" class="filter" :class="{active:filterIndex===i}" @tap="filterIndex=i">{{f}}</text></view>
      <view v-for="item in displayUsers" :key="item.id" class="user-card" :class="{boarding:boardingId===item.id}" @tap="openUser(item)">
        <image class="avatar" :src="item.avatar" mode="aspectFill"/>
        <view class="user-content">
          <view class="name-line"><text class="name">{{item.name}}</text><text class="role" :class="{purple:item.live}">{{item.live?'直播中':'交友'}}</text><text class="gender">♀ {{item.age}}</text></view>
          <text class="desc">{{item.desc}}</text>
          <view class="tags"><text v-for="tag in item.tags" :key="tag" class="tag"># {{tag}}</text></view>
          <view class="card-bottom"><text class="online"><i></i>{{item.online?'在线':'刚刚活跃'}}</text><text class="distance">{{item.distance}}</text><text v-if="item.live" class="enter" @tap.stop="enterRoom(item)">跟随进厅 ↗</text><text v-else class="like" @tap.stop="like(item)">♡ {{item.liked?'已喜欢':'喜欢'}}</text></view>
        </view>
      </view>
      <view class="loading">{{loading?'正在加载更多…': '已经到底啦，换个筛选继续发现'}}</view>
      <view class="safe-end"></view>
    </scroll-view>
    <view class="light-gate"><view class="gate-line"></view><text>MEET YOUR MOMENT</text></view>
  </view>

  <scroll-view v-else-if="activeTab===1" class="subpage" scroll-y>
    <view class="discover-hero"><text class="banner-kicker">EXPLORE MORE</text><text class="banner-title">发现精彩</text><text class="banner-sub">从兴趣开始，认识新的朋友</text></view>
    <view class="section-title">探索频道</view>
    <view class="channel-grid"><view v-for="(c,i) in channels" :key="c.name" class="channel" :class="'c'+i" @tap="openFeature('channel',c.name)"><text>{{c.icon}}</text><b>{{c.name}}</b><small>{{c.desc}}</small></view></view>
    <view class="section-title">今日话题</view><view v-for="topic in topics" :key="topic" class="topic" @tap="openFeature('topic',topic)"># {{topic}} <text>参与讨论 ›</text></view>
  </scroll-view>

  <scroll-view v-else-if="activeTab===2" class="subpage" scroll-y>
    <view class="message-head"><text class="banner-kicker">YOUR CONNECTIONS</text><text class="banner-title">消息</text><text class="banner-sub">每一次回应，都是新的开始</text></view>
    <view class="quick-actions"><view @tap="openFeature('likes')">♡<small>喜欢我的</small></view><view @tap="openFeature('visitors')">◉<small>最近访客</small></view><view @tap="openFeature('matches')">✦<small>新的匹配</small></view></view>
    <view class="section-title">最近聊天 <text class="muted">全部消息</text></view>
    <view v-for="(m,i) in chats" :key="m.name" class="chat-row" @tap="openChat(m)"><view class="chat-avatar">{{m.name.slice(0,1)}}</view><view class="chat-info"><b>{{m.name}}</b><small>{{m.last}}</small></view><view class="chat-meta"><small>{{m.time}}</small><i v-if="i===0">2</i></view></view>
  </scroll-view>

  <scroll-view v-else class="subpage" scroll-y>
    <view class="my-head"><view class="my-top"><text class="banner-kicker">MY SPACE</text><text @tap="openSettings">⚙ 设置</text></view><view class="my-profile"><view class="my-avatar">B</view><view><text class="my-name">遇见新朋友</text><small>完善资料，让更多人认识你</small></view><text class="edit" @tap="openAuth">编辑 ›</text></view><view class="completion"><view><text>资料完整度</text><b>65%</b></view><view class="progress"><i></i></view></view></view>
    <view class="my-stats"><view><b>0</b><small>获赞</small></view><view><b>0</b><small>关注</small></view><view><b>0</b><small>访客</small></view></view>
    <view class="menu"><view v-for="m in myMenus" :key="m.name" @tap="m.url?go(m.url):openFeature(m.type)"><text>{{m.icon}}　{{m.name}}</text><text>›</text></view></view>
  </scroll-view>

  <view class="tabbar">
    <view v-for="(t,i) in tabs" :key="t.label" class="tab" @tap="activeTab=i"><text class="tab-icon" :class="{selected:activeTab===i}">{{t.icon}}</text><text class="tab-label" :class="{active:activeTab===i}">{{t.label}}</text><i v-if="i===2" class="badge-dot"></i></view>
  </view>
  <view v-if="toast" class="toast" @tap="toast=''">{{toast}}</view>
</view>
</template>

<script setup>
import {ref} from 'vue'
const statusHeight=ref(0)
try { const info=uni.getSystemInfoSync(); statusHeight.value=info.statusBarHeight||0 } catch(e) { statusHeight.value=0 }
const activeTab=ref(0),refreshing=ref(false),loading=ref(false),toast=ref(''),boardingId=ref(''),filterIndex=ref(0)
const filters=['为你推荐','附近的人','刚刚活跃','兴趣相投']
const tabTitles=['推荐','发现','消息','我的']
const tabs=[{label:'推荐',icon:'⌂'},{label:'发现',icon:'✧'},{label:'消息',icon:'♡'},{label:'我的',icon:'◉'}]
const channels=[{icon:'♡',name:'同城交友',desc:'发现身边的缘分'},{icon:'♫',name:'兴趣社群',desc:'找到同好伙伴'},{icon:'◉',name:'语音派对',desc:'用声音认识彼此'},{icon:'✦',name:'直播互动',desc:'分享此刻精彩'}]
const topics=['周末去哪儿玩','分享最近喜欢的歌','理想中的第一次约会']
const myMenus=[{icon:'▧',name:'我的动态',type:'dynamic'},{icon:'♡',name:'我的喜欢',type:'likes'},{icon:'♧',name:'我的关注',type:'following'},{icon:'◇',name:'实名认证',type:'verify'},{icon:'⚙',name:'账号与设置',url:'/pages/settings/index'}]
const chats=ref([{name:'小鱼',last:'周末一起去看展吗？',time:'刚刚'},{name:'江姜',last:'哈哈哈哈太有意思了',time:'10:24'},{name:'柒',last:'欢迎来我的直播间～',time:'昨天'},{name:'安安',last:'你好，很高兴认识你',time:'周一'}])
const names=['浅安','小鱼','柒','江姜','早睡','优优','晚风','南栀','橘子海','林间','星野','桃桃','阿梨','七月','小满','可乐','山茶','知夏','月亮','初一','雾岛','一颗糖','晴天','木木','晚星','小鹿','青禾','橙子','云朵','念念']
const bios=['花自向阳开，慢慢成为更好的自己','喜欢旅行、摄影和路边的小猫','把普通日子过得闪闪发光','周末看展，偶尔徒步','认真生活，也认真认识新朋友','生活需要一点浪漫和惊喜']
const tags=[['旅行','摄影','咖啡'],['王者荣耀','电影','音乐'],['宠物','美食','徒步'],['阅读','看展','城市漫步']]
const users=ref(names.map((name,i)=>({id:i+1,name,age:20+i%8,avatar:'/static/avatar'+(i%6+1)+'.svg',desc:bios[i%bios.length],tags:tags[i%tags.length],online:i%3!==0,live:i%7===2,distance:(i+1)+' km',liked:false})))
const displayUsers=users
function openUser(item){uni.navigateTo({url:'/pages/user/index?name='+encodeURIComponent(item.name)})}
function enterRoom(item){uni.navigateTo({url:'/pages/room/index?name='+encodeURIComponent(item.name)})}
function openChat(item){uni.navigateTo({url:'/pages/chat/index?name='+encodeURIComponent(item.name)})}
function like(item){item.liked=!item.liked}
function go(url){uni.navigateTo({url})}
function openFeature(type,name=''){uni.navigateTo({url:'/pages/feature/index?type='+encodeURIComponent(type)+'&name='+encodeURIComponent(name)})}
function openSettings(){go('/pages/settings/index')}
function openAuth(){go('/pages/auth/index')}
function showNotice(){toast.value='暂时没有新通知'}
function showSearch(){openFeature('search')}
function refreshList(){refreshing.value=true;setTimeout(()=>{users.value=[...users.value].sort(()=>Math.random()-.5);refreshing.value=false;toast.value='推荐已刷新';setTimeout(()=>toast.value='',1500)},700)}
function loadMore(){if(loading.value)return;loading.value=true;setTimeout(()=>{const base=users.value.length;const extra=names.slice(0,10).map((name,i)=>({id:base+i+1,name:name+'·'+(Math.floor(base/10)+1),age:21+i%7,avatar:'/static/avatar'+(i%6+1)+'.svg',desc:bios[i%6],tags:tags[i%4],online:i%2===0,live:i%6===0,distance:(i+2)+' km',liked:false}));users.value.push(...extra);loading.value=false},800)}
function onScroll(){const query=uni.createSelectorQuery();query.selectAll('.user-card').boundingClientRect(rects=>{if(!rects||!rects.length)return;const target=uni.getSystemInfoSync().windowHeight*.48;const hit=rects.find(r=>r.top<target&&r.bottom>target);boardingId.value=hit?users.value[rects.indexOf(hit)]?.id:''}).exec()}
</script>

<style lang="scss" scoped>
/* Stable app shell: header, independently scrolling content and persistent tab bar */
.screen{position:fixed;inset:0;width:100%;height:100%;overflow:hidden;color:#302b3e;background:linear-gradient(145deg,#d9e9ff 0%,#f5d9f5 38%,#e6eaff 70%,#d8f4ff 100%);box-sizing:border-box}
.ambient{position:absolute;inset:0;pointer-events:none;opacity:.55;background:radial-gradient(ellipse at 75% 25%,#fff,transparent 42%),radial-gradient(ellipse at 5% 82%,#fff,transparent 38%)}
.safe-top{position:absolute;left:0;right:0;top:0;z-index:6}
.topbar{position:absolute;z-index:6;top:var(--safe-top,0px);left:0;right:0;height:88px;display:flex;align-items:center;justify-content:space-between;padding:0 6.5%;box-sizing:border-box}
.heading{font-size:36px;font-weight:1000;font-style:italic;letter-spacing:2px;text-shadow:1px 2px 1px #5a3f772e}
.actions{display:flex;align-items:center;gap:20px}.action-box{width:48px;height:50px;border-radius:17px;background:#ffffff70;box-shadow:0 5px 20px #fff5;display:flex;align-items:center;justify-content:center}.ticket{font-size:22px;color:#8b8cf3}.bell{font-size:25px;color:#f17db7}.search{font-size:40px;color:white;transform:rotate(-20deg)}
.feed-wrap,.subpage{position:absolute;left:0;right:0;top:calc(var(--safe-top,0px) + 88px);bottom:calc(70px + env(safe-area-inset-bottom,0px));min-height:0;box-sizing:border-box}
.feed{height:100%;padding:0 4.3%;box-sizing:border-box}
.banner,.discover-hero,.message-head{position:relative;display:flex;flex-direction:column;padding:18px 22px;margin-bottom:14px;border-radius:24px;background:linear-gradient(110deg,#ffffffb8,#ffffff58);border:1px solid #fff;box-shadow:0 8px 22px #a9a0c01c;box-sizing:border-box}
.banner-kicker{font-size:9px;letter-spacing:2px;color:#a38eb8}.banner-title{font-size:24px;font-weight:900;margin:7px 0 4px}.banner-sub{font-size:11px;color:#898294}.banner-orb{position:absolute;right:25px;top:25px;font-size:52px;color:#d88fc7;opacity:.6}
.filter-row{display:flex;gap:9px;overflow:hidden;margin:12px 0 16px}.filter{font-size:11px;padding:8px 13px;border-radius:18px;background:#ffffff75;color:#92899c;white-space:nowrap}.filter.active{background:#fff;color:#d36eae;box-shadow:0 3px 10px #d8b4d133}
.user-card{position:relative;display:flex;align-items:center;min-height:140px;margin:0 0 15px;padding:14px 14px 14px 17px;border-radius:42px;background:linear-gradient(110deg,#ffffffc4,#ffffff70);border:1.5px solid #fff;box-shadow:inset 0 1px 7px #fff9,0 8px 24px #ffffff35;box-sizing:border-box;transition:transform .22s,box-shadow .22s}
.user-card.boarding{transform:scale(1.012);border:2px solid #fff;box-shadow:0 0 0 2px #e8a6e8,0 0 18px 5px #d98be6aa,inset 0 0 15px #fff;animation:flowGlow 2.1s linear infinite}.user-card.boarding:after{content:'';position:absolute;inset:-3px;border-radius:45px;border:2px solid transparent;border-top-color:#fff;border-right-color:#cba8ff;filter:drop-shadow(0 0 5px #d17be9);pointer-events:none}
@keyframes flowGlow{50%{box-shadow:0 0 0 2px #b5c5ff,0 0 23px 7px #a98df0aa,inset 0 0 18px #fff}}
.avatar{width:clamp(70px,21vw,94px);height:clamp(70px,21vw,94px);flex-shrink:0;border-radius:50%;border:3px solid #fff;box-shadow:0 2px 8px #6b5d9029}.user-content{min-width:0;flex:1;margin-left:13px}.name-line{display:flex;align-items:center;gap:6px;white-space:nowrap}.name{max-width:48%;overflow:hidden;text-overflow:ellipsis;font-size:18px;font-weight:850}.role{font-size:10px;color:#fff;background:linear-gradient(100deg,#ff8fb8,#f4a0c5);border-radius:13px;padding:3px 7px}.role.purple{background:linear-gradient(100deg,#df7ce9,#b58cf4)}.gender{font-size:12px;color:#e879b6}.desc{display:block;color:#77717d;font-size:11px;margin-top:8px;white-space:nowrap;overflow:hidden;text-overflow:ellipsis}.tags{display:flex;gap:5px;margin-top:8px;flex-wrap:wrap}.tag{font-size:9px;color:#77717e;background:#ffffffb0;padding:4px 7px;border-radius:14px;white-space:nowrap}.card-bottom{display:flex;align-items:center;gap:9px;margin-top:9px;font-size:10px;color:#9c93a5}.online{display:flex;align-items:center;gap:4px}.online i{width:6px;height:6px;border-radius:50%;background:#70cfa7}.enter,.like{margin-left:auto;color:#d779b3}.enter{background:linear-gradient(100deg,#ff9ab8,#f06eaa);color:white;padding:6px 10px;border-radius:20px}
.loading{text-align:center;color:#a49bad;font-size:10px;padding:16px}.light-gate{position:absolute;left:5%;right:5%;top:48%;height:1px;pointer-events:none;z-index:4;opacity:.8}.gate-line{height:1px;background:linear-gradient(90deg,transparent,#fff,#d99bea,#fff,transparent);box-shadow:0 0 10px #d78ce9}.light-gate text{position:absolute;right:10px;top:-14px;font-size:7px;letter-spacing:2px;color:#b78dc5;background:#f3e7f5dd;padding:3px 6px;border-radius:10px}
.tabbar{position:absolute;z-index:10;left:0;right:0;bottom:0;height:calc(62px + env(safe-area-inset-bottom,0px));padding:7px 3% env(safe-area-inset-bottom,0px);box-sizing:border-box;display:flex;justify-content:space-around;align-items:center;background:#fffffffa;border-top:1px solid #fff;box-shadow:0 -5px 22px #b6a8c02a}.tab{width:25%;height:100%;display:flex;flex-direction:column;align-items:center;justify-content:center;gap:2px;position:relative}.tab-icon{font-size:25px;line-height:30px;color:#c8c6d1}.tab-icon.selected{color:#df75bb;text-shadow:0 2px 3px #e8b8e1}.tab-label{font-size:10px;color:#c3c0ca}.tab-label.active{color:#e47fba}.badge-dot{position:absolute;top:5px;right:28%;width:6px;height:6px;background:#f06e8d;border-radius:50%}
.subpage{padding:8px 17px 18px;overflow-y:auto;-webkit-overflow-scrolling:touch}.discover-hero{min-height:120px;justify-content:center}.section-title{font-size:16px;font-weight:800;margin:20px 3px 12px}.channel-grid{display:grid;grid-template-columns:1fr 1fr;gap:11px}.channel{height:108px;border-radius:20px;padding:14px;background:#ffffffbd;display:flex;flex-direction:column;justify-content:space-between;box-sizing:border-box}.channel text{font-size:24px;color:#d781ba}.channel b{font-size:14px}.channel small{font-size:10px;color:#999}.c1 text{color:#9a9ce8}.c2 text{color:#8ec9c6}.c3 text{color:#e8a17d}.topic{padding:17px;background:#ffffffbd;border-radius:15px;margin:9px 0;font-size:13px;color:#9d73ad}.topic text{float:right;color:#aaa;font-size:10px}.quick-actions{display:flex;justify-content:space-around;padding:17px 5px;background:#ffffffbd;border-radius:18px;color:#d47db4;font-size:22px}.quick-actions view{text-align:center}.quick-actions small{display:block;font-size:10px;color:#92899c;margin-top:7px}.muted{float:right;color:#aaa;font-size:10px;font-weight:400}.chat-row{display:flex;align-items:center;padding:13px 5px;border-bottom:1px solid #ffffffa8;gap:12px}.chat-avatar,.my-avatar{width:46px;height:46px;border-radius:50%;background:linear-gradient(140deg,#c5dfff,#f3c7e6);display:flex;align-items:center;justify-content:center;color:#9a78b1;font-weight:800;flex-shrink:0}.chat-info{flex:1;min-width:0}.chat-info b,.chat-info small{display:block}.chat-info b{font-size:13px}.chat-info small{font-size:10px;color:#999;margin-top:5px;white-space:nowrap;overflow:hidden;text-overflow:ellipsis}.chat-meta{display:flex;flex-direction:column;align-items:flex-end;gap:6px;color:#aaa;font-size:9px}.chat-meta i{font-style:normal;background:#ee8db6;color:white;border-radius:10px;padding:2px 6px}
.my-head{margin:0 -17px;padding:20px 22px 25px;background:linear-gradient(120deg,#d9e9ff,#f4d9f0);border-radius:0 0 26px 26px}.my-top{display:flex;justify-content:space-between;font-size:11px;color:#9a87aa}.my-profile{display:flex;align-items:center;gap:12px;margin-top:25px}.my-avatar{width:62px;height:62px;border:3px solid white;font-size:22px}.my-name{display:block;font-size:18px;font-weight:900}.my-profile small{display:block;font-size:10px;color:#888;margin-top:5px}.edit{margin-left:auto;color:#aa80b5;font-size:11px}.completion{margin-top:20px;font-size:10px;color:#8c8196}.completion b{float:right}.progress{height:5px;border-radius:5px;background:#ffffffaa;margin-top:8px}.progress i{display:block;width:65%;height:100%;border-radius:5px;background:linear-gradient(90deg,#ed9bc5,#a89bf0)}.my-stats{display:flex;justify-content:space-around;padding:20px 0;background:#ffffffbd;border-radius:18px;margin-top:14px}.my-stats b,.my-stats small{display:block;text-align:center}.my-stats b{font-size:18px}.my-stats small{font-size:10px;color:#999;margin-top:4px}.menu{background:#ffffffc7;border-radius:18px;margin-top:15px;padding:0 16px}.menu view{height:51px;display:flex;justify-content:space-between;align-items:center;border-bottom:1px solid #f3edf5;font-size:12px}.menu view:last-child{border:0}.menu view text:last-child{font-size:20px;color:#bbb}
.toast{position:absolute;z-index:20;bottom:calc(82px + env(safe-area-inset-bottom,0px));left:50%;transform:translateX(-50%);max-width:85%;white-space:nowrap;background:#282337e8;color:white;padding:10px 18px;border-radius:22px;font-size:12px}.safe-end{height:12px}
</style>