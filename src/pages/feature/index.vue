<template>
  <view class="page">
    <view class="nav"><text class="back" @tap="back">‹</text><text class="nav-title">{{title}}</text><text class="nav-action" @tap="save">{{actionLabel}}</text></view>
    <view v-if="type==='search'" class="search-box"><input v-model="keyword" placeholder="搜索昵称、兴趣、城市" confirm-type="search"/><text @tap="search">搜索</text></view>
    <view v-if="type==='edit'" class="card">
      <view class="avatar-edit" @tap="toast='头像选择：演示数据已就绪'">B<small>更换头像</small></view>
      <label>昵称</label><input v-model="profile.nickname" placeholder="请输入昵称"/>
      <label>所在城市</label><input v-model="profile.city" placeholder="例如：杭州"/>
      <label>个人介绍</label><textarea v-model="profile.bio" maxlength="120" placeholder="介绍一下自己，让大家更了解你"/>
      <label>兴趣标签（用空格分隔）</label><input v-model="profile.tags" placeholder="旅行 摄影 咖啡"/>
      <button class="primary" @tap="save">保存资料</button>
    </view>
    <view v-else-if="type==='dynamic'" class="card">
      <textarea v-model="postText" placeholder="记录此刻的心情、生活和灵感..." maxlength="300"/>
      <view class="attach" @tap="toast='图片选择已打开（演示模式）'">＋ 添加照片</view>
      <button class="primary" @tap="publish">发布动态</button>
      <view class="section">我的动态</view>
      <view v-for="(p,i) in posts" :key="i" class="post"><text>{{p.text}}</text><small>{{p.time}} · ♡ {{p.likes}}</small></view>
    </view>
    <view v-else-if="type==='verify'" class="card">
      <view class="verify-icon">✓</view><view class="center-title">完成身份认证</view><view class="muted">认证后可获得专属标识，提升资料可信度</view>
      <label>真实姓名</label><input v-model="realName" placeholder="请输入真实姓名"/>
      <label>身份证号码</label><input v-model="idNumber" placeholder="请输入身份证号码"/>
      <button class="primary" @tap="verify">提交认证资料</button><small class="muted">演示环境仅做格式校验，不会上传或验证真实身份</small>
    </view>
    <view v-else-if="type==='search'" class="list">
      <view class="section">推荐用户</view><view v-for="u in filteredUsers" :key="u" class="person" @tap="openUser(u)"><view class="avatar">{{u.slice(0,1)}}</view><view class="person-info"><b>{{u}}</b><small>兴趣相投 · 欢迎认识</small></view><text>查看 ›</text></view>
      <view v-if="!filteredUsers.length" class="empty">没有找到匹配的人，试试其他关键词</view>
    </view>
    <view v-else class="list">
      <view v-if="type==='privacy'" class="card"><view class="setting-row"><view><b>显示在线状态</b><small>允许其他人看到你是否在线</small></view><switch :checked="toggles.online" @change="v=>toggle('online',v.detail.value)"/></view><view class="setting-row"><view><b>允许陌生人私信</b><small>关闭后仅互相关注的人可以联系你</small></view><switch :checked="toggles.dm" @change="v=>toggle('dm',v.detail.value)"/></view><view class="setting-row"><view><b>显示距离</b><small>在个人资料中展示大致距离</small></view><switch :checked="toggles.distance" @change="v=>toggle('distance',v.detail.value)"/></view></view>
      <view v-else-if="type==='notifications'||type==='general'" class="card"><view v-for="item in optionItems" :key="item.key" class="setting-row"><view><b>{{item.title}}</b><small>{{item.desc}}</small></view><switch :checked="toggles[item.key]" @change="v=>toggle(item.key,v.detail.value)"/></view></view>
      <view v-else-if="type==='account'" class="card"><view class="info-row"><text>手机号</text><b>未绑定</b></view><view class="info-row"><text>登录密码</text><b @tap="toast='请通过登录页的忘记密码功能重设'">修改 ›</b></view><view class="info-row"><text>账号状态</text><b>正常</b></view><view class="info-row"><text>注销账号</text><b class="danger" @tap="confirmDelete">申请注销 ›</b></view></view>
      <view v-else-if="type==='blacklist'" class="card"><view v-for="(person,i) in blocked" :key="person" class="person"><view class="avatar">{{person.slice(0,1)}}</view><view class="person-info"><b>{{person}}</b><small>已加入黑名单</small></view><text class="action" @tap="unblock(i)">移除</text></view><view v-if="!blocked.length" class="empty">黑名单为空，你可以安心认识新朋友</view></view>
      <view v-else-if="type==='about'||type==='agreement'" class="card legal"><view class="brand">遇见 MEET</view><b>{{type==='about'?'让每一次相遇都值得期待':'用户协议与隐私说明'}}</b><p>{{type==='about'?'遇见是一款帮助用户发现兴趣伙伴、分享生活和建立真实连接的社交产品。当前版本为可交互产品原型，账号、消息、音视频和直播服务尚未连接生产后端。':'请遵守法律法规与社区规范，尊重其他用户的意愿，不发布违法、骚扰、欺诈或侵犯他人权益的内容。个人资料和互动数据在当前演示版本中保存在本地设备。真实服务上线前需配置正式隐私政策、数据处理规则和用户申诉渠道。'}}</p><p>版本号：1.0.1</p></view>
      <view v-else class="card"><view class="empty">暂无内容</view></view>
    </view>
    <view v-if="toast" class="toast" @tap="toast=''">{{toast}}</view>
  </view>
</template>
<script setup>
import {ref,computed} from 'vue'
const type=ref(''),title=ref('功能'),toast=ref(''),keyword=ref(''),postText=ref(''),realName=ref(''),idNumber=ref('')
const actionLabel=ref('')
const profile=ref({nickname:'遇见新朋友',city:'杭州',bio:'认真生活，慢慢遇见。',tags:'旅行 摄影 咖啡'})
const allUsers=['浅安','小鱼','柒','江姜','早睡','优优','晚风','南栀','橘子海','林间','星野','桃桃']
const filteredUsers=computed(()=>allUsers.filter(x=>!keyword.value||x.includes(keyword.value)))
const posts=ref([{text:'生活碎片：今天也要保持好奇心 ☀️',time:'今天 09:30',likes:23},{text:'周末的咖啡和一本好书。',time:'昨天 18:20',likes:16}])
const blocked=ref(['测试用户A','测试用户B'])
const toggles=ref({online:true,dm:true,distance:true,likes:true,messages:true,system:true,vibration:false,sound:true})
const optionItems=computed(()=>type.value==='notifications'?[{key:'likes',title:'喜欢与关注',desc:'收到喜欢、关注时通知'},{key:'messages',title:'新消息提醒',desc:'收到私信时通知'},{key:'system',title:'系统通知',desc:'活动与安全提醒'}]:[{key:'sound',title:'提示音',desc:'播放消息提示音'},{key:'vibration',title:'振动反馈',desc:'收到消息时振动'}])
const titles={search:'搜索',edit:'编辑资料',dynamic:'我的动态',verify:'实名认证',privacy:'隐私设置',notifications:'消息通知',general:'通用设置',account:'账号与安全',blacklist:'黑名单',about:'关于我们',agreement:'用户协议'}
function onLoad(query){type.value=query.type||'about';title.value=titles[type.value]||'功能';actionLabel.value=type.value==='edit'?'保存':'';try{const p=uni.getStorageSync('meet-profile');if(p)profile.value=p;const t=uni.getStorageSync('meet-settings');if(t)toggles.value={...toggles.value,...t};const b=uni.getStorageSync('meet-blacklist');if(b)blocked.value=b;const ps=uni.getStorageSync('meet-posts');if(ps)posts.value=ps}catch(e){}}
onLoad(uni.getCurrentPages().slice(-1)[0]?.options||{})
function back(){uni.navigateBack()}
function toggle(k,v){toggles.value[k]=v;uni.setStorageSync('meet-settings',toggles.value)}
function save(){uni.setStorageSync('meet-profile',profile.value);toast.value='资料已保存在本机'}
function publish(){if(!postText.value.trim()){toast.value='先写下你的动态内容';return}posts.value.unshift({text:postText.value.trim(),time:'刚刚',likes:0});uni.setStorageSync('meet-posts',posts.value);postText.value='';toast.value='动态发布成功'}
function verify(){if(!realName.value.trim()||idNumber.value.trim().length<8){toast.value='请填写姓名和有效证件号码';return}toast.value='资料格式已通过演示校验，正式认证需接入服务端'}
function search(){if(!keyword.value.trim())toast.value='请输入搜索关键词'}
function openUser(u){uni.navigateTo({url:'/pages/user/index?name='+encodeURIComponent(u)})}
function unblock(i){blocked.value.splice(i,1);uni.setStorageSync('meet-blacklist',blocked.value);toast.value='已移出黑名单'}
function confirmDelete(){toast.value='注销申请需完成身份验证后提交'}
</script>
<style scoped lang="scss">
.page{min-height:100vh;background:linear-gradient(150deg,#e5efff,#f8e9f4 55%,#eef6ff);color:#342e40;padding:env(safe-area-inset-top) 16px 30px;box-sizing:border-box}.nav{height:58px;display:flex;align-items:center;gap:16px}.back{font-size:30px;color:#a47eb6}.nav-title{font-size:18px;font-weight:800;flex:1}.nav-action{font-size:12px;color:#c96da9}.card,.list{background:#ffffffc9;border:1px solid #fff;border-radius:22px;padding:18px;margin-top:12px;box-shadow:0 10px 28px #a99ab51a}.search-box{display:flex;gap:10px;align-items:center;background:#fff;border-radius:16px;padding:8px 14px;margin:12px 0}.search-box input{flex:1}.search-box text,.action{color:#d46eaa}.avatar-edit{width:78px;height:78px;border-radius:50%;background:linear-gradient(130deg,#d3e4ff,#f5cce7);margin:5px auto 22px;display:flex;flex-direction:column;align-items:center;justify-content:center;font-size:24px;color:#9d79b2}.avatar-edit small{font-size:9px}label{display:block;font-size:12px;color:#8e8498;margin:14px 0 6px}input,textarea{width:100%;box-sizing:border-box;background:#f7f4fa;border-radius:12px;padding:12px;font-size:13px}textarea{min-height:110px}.primary{width:100%;margin-top:18px;border:0;border-radius:24px;background:linear-gradient(100deg,#f39bc4,#a99af0);color:white;font-size:14px}.section{font-weight:800;margin:20px 0 12px}.post{padding:14px 0;border-bottom:1px solid #f0ebf3;font-size:13px}.post small{display:block;color:#aaa;font-size:10px;margin-top:8px}.attach{padding:12px;background:#f7f3fa;border-radius:12px;color:#a77bb2;font-size:12px;margin-top:10px}.verify-icon{width:62px;height:62px;border-radius:50%;display:grid;place-items:center;background:#e5f6ed;color:#51b783;font-size:30px;margin:10px auto}.center-title{text-align:center;font-size:18px;font-weight:800}.muted{display:block;text-align:center;color:#a39aa9;font-size:11px;margin:10px 0 18px}.setting-row,.info-row{display:flex;align-items:center;justify-content:space-between;gap:12px;padding:15px 0;border-bottom:1px solid #f1edf4}.setting-row:last-child,.info-row:last-child{border:0}.setting-row b,.setting-row small{display:block}.setting-row b,.info-row{font-size:13px}.setting-row small{font-size:10px;color:#aaa;margin-top:5px}.info-row text{color:#8f8797}.info-row b{font-weight:500}.danger{color:#df7099}.person{display:flex;align-items:center;gap:12px;padding:12px 0;border-bottom:1px solid #f1edf4}.avatar{width:42px;height:42px;border-radius:50%;background:linear-gradient(130deg,#d3e4ff,#f5cce7);display:grid;place-items:center;color:#9879b1}.person-info{flex:1}.person-info b,.person-info small{display:block}.person-info small{font-size:10px;color:#aaa;margin-top:4px}.person>text{font-size:12px}.empty{text-align:center;color:#a49baa;padding:32px 10px;font-size:12px}.brand{font-size:21px;font-weight:900;color:#bd79b4;margin-bottom:20px}.legal b{font-size:15px}.legal p{font-size:12px;line-height:1.9;color:#777;margin-top:14px}.toast{position:fixed;bottom:30px;left:10%;width:80%;text-align:center;background:#30283de8;color:white;padding:11px;border-radius:24px;font-size:12px}
</style>