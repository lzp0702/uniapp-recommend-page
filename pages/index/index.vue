<template>
  <view class="screen">
    <view class="ambient"></view>
    <view class="status">
      <text class="clock">15:22</text>
      <view class="status-right"><text>▮▮▮</text><text class="wifi">◔</text><text class="battery">88%</text></view>
    </view>
    <view class="topbar">
      <text class="heading">推荐</text>
      <view class="actions">
        <view class="action-box"><text class="ticket">★</text></view>
        <view class="action-box"><text class="bell">♟</text></view>
        <text class="search">⌕</text>
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
 {name:'c.浅安',role:'交友',age:'21',avatar:'/static/avatar1.jpg',desc:'',tags:[],serif:true},
 {name:'QY.小鱼',role:'交友',age:'21',avatar:'/static/avatar2.jpg',desc:'花自向阳开 我比你想象中要坚韧',tags:['王者荣耀','cosplay','steam'],serif:true},
 {name:'柒',role:'个播',age:'',avatar:'/static/avatar3.jpg',desc:'',tags:['摄影','素食主义','旅行'],live:true},
 {name:'江姜',role:'个播',age:'24',avatar:'/static/avatar4.jpg',desc:'没有个性',tags:['看电影','萌宠','动漫']},
 {name:'H.早睡',role:'个播',age:'25',avatar:'/static/avatar5.jpg',desc:'压抑的情绪、压抑的生活',tags:['睡觉','王者荣耀'],serif:true},
 {name:'优优💕优乐美',role:'个播',age:'20',avatar:'/static/avatar6.jpg',desc:'幸福就是 你们在我身边',tags:['看电影','王者荣耀','追剧']}
])
const wave=[18,28,14,34,22,42,20,32,16,38,24,44,19,31,15,26,36,18,29,13,22,34]
const tabs=[{label:'首页',icon:'⌂'},{label:'娱乐',icon:'☻'},{label:'消息',icon:'☷'},{label:'我',icon:'♧'}]
const activeTab=ref(0), toast=ref('')
function enterRoom(item){ toast.value=`正在进入${item.name}的直播间`; setTimeout(()=>toast.value='',1600) }
</script>

<style lang="scss" scoped>
.screen{position:relative;height:100vh;min-height:620px;overflow:hidden;color:#24212e;background:linear-gradient(145deg,#d9e9ff 0%,#f5d9f5 36%,#e6eaff 68%,#d8f4ff 100%);}
.ambient{position:absolute;inset:0;pointer-events:none;opacity:.65;background:repeating-linear-gradient(132deg,rgba(255,255,255,.24) 0,rgba(255,255,255,.24) 1px,transparent 1px,transparent 8px),radial-gradient(ellipse at 75% 28%,rgba(255,255,255,.9),transparent 40%),radial-gradient(ellipse at 5% 82%,rgba(255,255,255,.7),transparent 38%);}
.status{height:calc(env(safe-area-inset-top) + 34px);padding:env(safe-area-inset-top) 8.5% 0;display:flex;align-items:center;justify-content:space-between;position:relative;z-index:1;color:#080811;font-weight:700;}
.clock{font-size:20px}.status-right{display:flex;align-items:center;gap:9px;font-size:16px}.wifi{font-size:22px}.battery{background:#38bd68;color:white;border-radius:5px;padding:1px 5px;font-size:14px}
.topbar{height:100px;display:flex;align-items:center;justify-content:space-between;padding:0 6.5%;position:relative;z-index:1}
.heading{font-size:38px;font-weight:1000;font-style:italic;letter-spacing:2px;text-shadow:1px 2px 1px rgba(90,63,119,.18)}
.actions{display:flex;align-items:center;gap:24px}.action-box{width:54px;height:58px;border-radius:18px;background:rgba(255,255,255,.28);box-shadow:0 5px 20px rgba(255,255,255,.35);display:flex;align-items:center;justify-content:center}.ticket{font-size:22px;color:white;background:linear-gradient(140deg,#9edbff,#8b8cf3);padding:7px 10px;border-radius:7px}.bell{font-size:26px;color:#f17db7;text-shadow:0 2px 7px #f5a8d3}.search{font-size:43px;color:white;line-height:1;transform:rotate(-20deg)}
.feed{position:relative;z-index:1;height:calc(100vh - env(safe-area-inset-top) - 34px - 100px - 104px);padding:0 4.3%;}
.user-card{position:relative;display:flex;align-items:center;min-height:154px;margin:0 0 22px;padding:18px 20px 18px 28px;border-radius:62px;background:linear-gradient(110deg,rgba(255,255,255,.58),rgba(255,255,255,.28));border:1.5px solid rgba(255,255,255,.78);box-shadow:inset 0 1px 7px rgba(255,255,255,.8),0 8px 24px rgba(255,255,255,.18);backdrop-filter:blur(12px);}
.avatar{width:110px;height:110px;flex-shrink:0;border-radius:50%;border:3px solid rgba(255,255,255,.95);box-shadow:0 2px 8px rgba(107,93,144,.16);}
.user-content{min-width:0;flex:1;margin-left:16px}.name-line{display:flex;align-items:center;gap:7px;white-space:nowrap}.name{font-size:24px;font-weight:800;letter-spacing:.2px}.name.serif{font-family:Georgia,serif;font-style:italic}.role{font-size:15px;font-weight:800;color:#fff;background:linear-gradient(100deg,#ff8fb8,#f4a0c5);border-radius:13px;padding:3px 10px}.role.purple{background:linear-gradient(100deg,#df7ce9,#b58cf4)}.gender{font-size:18px;color:#e879b6;font-weight:600}.desc{display:block;color:#77717d;font-size:17px;margin-top:11px;white-space:nowrap;overflow:hidden;text-overflow:ellipsis}.tags{display:flex;gap:8px;margin-top:12px;flex-wrap:wrap}.tag{font-size:14px;color:#77717e;background:rgba(255,255,255,.48);padding:4px 9px;border-radius:14px;white-space:nowrap}.hash{color:#e77db7;margin-right:3px}
.user-card.live{min-height:178px;margin-top:0;border:2px solid rgba(255,255,255,.95);box-shadow:inset 0 0 10px rgba(255,255,255,.9),0 0 17px rgba(255,255,255,.7);padding-top:20px;padding-bottom:18px}.live-bottom{height:39px;display:flex;align-items:center;position:relative;margin-top:7px}.wave{height:35px;display:flex;align-items:flex-end;gap:4px;opacity:.72}.wave i{width:4px;border-radius:4px;background:#c59af4}.pause{margin-left:auto;margin-right:10px;color:#df8cc9;font-weight:900;font-size:18px}.enter{background:linear-gradient(100deg,#ff9ab8,#f06eaa);color:white;font-size:16px;padding:9px 18px;border-radius:24px;box-shadow:0 3px 9px rgba(239,116,177,.22)}
.safe-end{height:22px}
.tabbar{position:absolute;z-index:3;bottom:0;left:0;right:0;height:calc(104px + env(safe-area-inset-bottom));padding:10px 3% env(safe-area-inset-bottom);display:flex;justify-content:space-around;background:rgba(255,255,255,.96);box-shadow:0 -5px 22px rgba(255,255,255,.38)}.tab{width:25%;display:flex;flex-direction:column;align-items:center;justify-content:flex-start;gap:6px}.tab-icon{height:48px;position:relative;color:#c8c6d1;font-size:34px;line-height:44px}.tab-icon.selected{color:#df75bb;text-shadow:0 2px 3px #e8b8e1}.tab-label{font-size:17px;color:#c3c0ca}.tab-label.active{color:#e47fba}.count{position:absolute;top:-3px;left:25px;background:#f05c68;color:white;border-radius:14px;font-size:12px;line-height:18px;padding:0 5px;border:1px solid white}
.toast{position:absolute;z-index:5;bottom:130px;left:50%;transform:translateX(-50%);background:rgba(40,35,55,.78);color:white;padding:10px 18px;border-radius:22px;font-size:14px}
@media (max-height:700px){.user-card{min-height:130px;margin-bottom:12px}.avatar{width:90px;height:90px}.user-card.live{min-height:150px}.name{font-size:21px}.desc{font-size:14px}.tag{font-size:12px}.feed{height:calc(100vh - env(safe-area-inset-top) - 34px - 85px - 90px)}.topbar{height:85px}.tabbar{height:90px}}
</style>