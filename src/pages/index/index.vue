<template>
  <view class="screen">
    <view class="ambient"></view>

    <!-- 参考 Vant NavBar：固定高度、左右插槽、中间标题；不绘制模拟系统状态栏 -->
    <view class="navbar">
      <view class="nav-left">
        <text class="nav-mark">推荐</text>
      </view>
      <view class="nav-title">推荐</view>
      <view class="nav-right">
        <view class="nav-action ticket-action"><text>★</text></view>
        <view class="nav-action notice-action"><text>♟</text></view>
        <view class="search-action">⌕</view>
      </view>
    </view>

    <scroll-view class="feed" scroll-y :show-scrollbar="false">
      <view v-for="(item,index) in users" :key="item.name" class="user-card" :class="{live:item.live}">
        <image class="avatar" :src="item.avatar" mode="aspectFill" />
        <view class="user-content">
          <view class="name-line">
            <text class="name" :class="{serif:item.serif}">{{item.name}}</text>
            <text class="role" :class="{purple:item.live}">{{item.role}}</text>
            <text class="gender">♀{{item.age || ''}}</text>
          </view>
          <text v-if="item.desc" class="desc">{{item.desc}}</text>
          <view v-if="item.tags.length" class="tags">
            <text v-for="tag in item.tags" :key="tag" class="tag"><text class="hash">#</text>{{tag}}</text>
          </view>
          <view v-if="item.live" class="live-bottom">
            <view class="wave"><i v-for="(h,n) in wave" :key="n" :style="{height:h+'rpx'}"></i></view>
            <view class="pause">Ⅱ</view>
            <view class="enter" @tap="enterRoom(item)">跟随进厅</view>
          </view>
        </view>
      </view>
      <view class="safe-end"></view>
    </scroll-view>

    <view class="tabbar">
      <view v-for="(tab,i) in tabs" :key="tab.label" class="tab" @tap="activeTab=i">
        <view class="tab-icon" :class="{selected:activeTab===i}">
          <text>{{tab.icon}}</text>
          <text v-if="i===2" class="count">99+</text>
        </view>
        <text class="tab-label" :class="{active:activeTab===i}">{{tab.label}}</text>
      </view>
    </view>
    <view v-if="toast" class="toast">{{toast}}</view>
  </view>
</template>

<script setup>
import { ref } from 'vue'
const users = ref([
 {name:'c.浅安',role:'交友',age:'21',avatar:'/static/avatar1.svg',desc:'',tags:[],serif:true},
 {name:'QY.小鱼',role:'交友',age:'21',avatar:'/static/avatar2.svg',desc:'花自向阳开 我比你想象中要坚韧',tags:['王者荣耀','cosplay','steam'],serif:true},
 {name:'柒',role:'个播',age:'',avatar:'/static/avatar3.svg',desc:'',tags:['摄影','素食主义','旅行'],live:true},
 {name:'江姜',role:'个播',age:'24',avatar:'/static/avatar4.svg',desc:'没有个性',tags:['看电影','萌宠','动漫']},
 {name:'H.早睡',role:'个播',age:'25',avatar:'/static/avatar5.svg',desc:'压抑的情绪、压抑的生活',tags:['睡觉','王者荣耀'],serif:true},
 {name:'优优💕优乐美',role:'个播',age:'20',avatar:'/static/avatar6.svg',desc:'幸福就是 你们在我身边',tags:['看电影','王者荣耀','追剧']}
])
const wave=[18,28,14,34,22,42,20,32,16,38,24,44,19,31,15,26,36,18,29,13,22,34]
const tabs=[{label:'首页',icon:'⌂'},{label:'娱乐',icon:'☻'},{label:'消息',icon:'☷'},{label:'我',icon:'♧'}]
const activeTab=ref(0), toast=ref('')
function enterRoom(item){ toast.value=`正在进入${item.name}的直播间`; setTimeout(()=>toast.value='',1600) }
</script>

<style lang="scss" scoped>
.screen {
  position: relative;
  display: flex;
  flex-direction: column;
  width: 100%;
  height: 100vh;
  height: 100dvh;
  min-height: 0;
  overflow: hidden;
  color: #24212e;
  background: linear-gradient(145deg,#d9e9ff 0%,#f5d9f5 36%,#e6eaff 68%,#d8f4ff 100%);
  box-sizing: border-box;
}
.ambient {
  position: absolute;
  inset: 0;
  pointer-events: none;
  opacity: .65;
  background: repeating-linear-gradient(132deg,rgba(255,255,255,.24) 0,rgba(255,255,255,.24) 1px,transparent 1px,transparent 8px),radial-gradient(ellipse at 75% 28%,rgba(255,255,255,.9),transparent 40%),radial-gradient(ellipse at 5% 82%,rgba(255,255,255,.7),transparent 38%);
}
/* Vant NavBar 的核心尺寸与左右插槽布局 */
.navbar {
  position: relative;
  z-index: 2;
  flex: 0 0 46px;
  height: 46px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 16px;
  box-sizing: border-box;
  background: rgba(255,255,255,.18);
  border-bottom: 1px solid rgba(255,255,255,.42);
  backdrop-filter: blur(14px);
}
.nav-left,.nav-right { display:flex; align-items:center; min-width:72px; height:100%; }
.nav-left { justify-content:flex-start; }
.nav-right { justify-content:flex-end; gap:12px; }
.nav-mark { font-size:14px; color:rgba(74,66,91,.72); }
.nav-title { position:absolute; left:50%; top:50%; transform:translate(-50%,-50%); max-width:55%; overflow:hidden; text-overflow:ellipsis; white-space:nowrap; font-size:17px; line-height:24px; font-weight:600; color:#282435; }
.nav-action { width:30px; height:30px; display:flex; align-items:center; justify-content:center; border-radius:50%; background:rgba(255,255,255,.55); box-shadow:0 2px 8px rgba(91,79,132,.08); }
.ticket-action { color:#8f8be9; font-size:16px; }
.notice-action { color:#ed8abd; font-size:17px; }
.search-action { width:26px; text-align:center; color:#716b82; font-size:30px; line-height:30px; transform:rotate(-20deg); }
.feed {
  position: relative;
  z-index: 1;
  flex: 1 1 0;
  min-height: 0;
  width: 100%;
  box-sizing: border-box;
  padding: 14px 4.3% 0;
}
.user-card {
  display:flex;
  align-items:center;
  min-height:154px;
  margin:0 0 18px;
  padding:16px 16px 16px 20px;
  box-sizing:border-box;
  border-radius:62px;
  background:linear-gradient(110deg,rgba(255,255,255,.68),rgba(255,255,255,.34));
  border:1.5px solid rgba(255,255,255,.82);
  box-shadow:inset 0 1px 7px rgba(255,255,255,.8),0 8px 24px rgba(255,255,255,.18);
  backdrop-filter:blur(12px);
}
.avatar { width:clamp(76px, 23vw, 110px); height:clamp(76px, 23vw, 110px); flex-shrink:0; border-radius:50%; border:3px solid rgba(255,255,255,.95); box-shadow:0 2px 8px rgba(107,93,144,.16); box-sizing:border-box; }
.user-content { min-width:0; flex:1; margin-left:14px; }
.name-line { display:flex; align-items:center; gap:6px; min-width:0; white-space:nowrap; }
.name { max-width:55%; overflow:hidden; text-overflow:ellipsis; font-size:clamp(17px,5vw,23px); font-weight:800; letter-spacing:.2px; }
.name.serif { font-family:Georgia,serif; font-style:italic; }
.role { flex-shrink:0; font-size:12px; font-weight:800; color:#fff; background:linear-gradient(100deg,#ff8fb8,#f4a0c5); border-radius:13px; padding:3px 8px; }
.role.purple { background:linear-gradient(100deg,#df7ce9,#b58cf4); }
.gender { flex-shrink:0; font-size:14px; color:#e879b6; font-weight:600; }
.desc { display:block; color:#77717d; font-size:13px; line-height:19px; margin-top:9px; white-space:nowrap; overflow:hidden; text-overflow:ellipsis; }
.tags { display:flex; gap:6px; margin-top:10px; flex-wrap:wrap; }
.tag { font-size:11px; line-height:18px; color:#77717e; background:rgba(255,255,255,.58); padding:2px 7px; border-radius:14px; white-space:nowrap; }
.hash { color:#e77db7; margin-right:2px; }
.user-card.live { min-height:170px; border:2px solid rgba(255,255,255,.95); box-shadow:inset 0 0 10px rgba(255,255,255,.9),0 0 17px rgba(255,255,255,.7); }
.live-bottom { height:34px; display:flex; align-items:center; margin-top:6px; }
.wave { height:30px; display:flex; align-items:flex-end; gap:3px; opacity:.72; overflow:hidden; }
.wave i { width:3px; border-radius:4px; background:#c59af4; }
.pause { margin-left:auto; margin-right:8px; color:#df8cc9; font-weight:900; font-size:16px; }
.enter { flex-shrink:0; background:linear-gradient(100deg,#ff9ab8,#f06eaa); color:white; font-size:13px; padding:8px 13px; border-radius:24px; box-shadow:0 3px 9px rgba(239,116,177,.22); }
.safe-end { height:12px; }
.tabbar {
  position:relative;
  z-index:3;
  flex:0 0 calc(62px + env(safe-area-inset-bottom));
  min-height:62px;
  padding:8px 3% env(safe-area-inset-bottom);
  box-sizing:content-box;
  display:flex;
  justify-content:space-around;
  background:rgba(255,255,255,.96);
  box-shadow:0 -5px 22px rgba(255,255,255,.38);
}
.tab { width:25%; display:flex; flex-direction:column; align-items:center; justify-content:flex-start; gap:2px; }
.tab-icon { height:34px; position:relative; color:#c8c6d1; font-size:28px; line-height:34px; }
.tab-icon.selected { color:#df75bb; text-shadow:0 2px 3px #e8b8e1; }
.tab-label { font-size:12px; line-height:18px; color:#c3c0ca; }
.tab-label.active { color:#e47fba; }
.count { position:absolute; top:-2px; left:19px; background:#f05c68; color:white; border-radius:14px; font-size:9px; line-height:14px; padding:0 4px; border:1px solid white; }
.toast { position:absolute; z-index:5; bottom:calc(82px + env(safe-area-inset-bottom)); left:50%; transform:translateX(-50%); max-width:80%; white-space:nowrap; background:rgba(40,35,55,.78); color:white; padding:10px 18px; border-radius:22px; font-size:13px; }
@media (max-height:700px) {
  .feed { padding-top:10px; }
  .user-card { min-height:124px; margin-bottom:10px; padding-top:12px; padding-bottom:12px; }
  .avatar { width:76px; height:76px; }
  .user-card.live { min-height:145px; }
  .desc { margin-top:5px; }
  .tags { margin-top:6px; }
}
</style>
