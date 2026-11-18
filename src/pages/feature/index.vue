<template>
  <view class="page">
    <view class="nav"><text class="back" @tap="back">‹</text><text class="nav-title">{{title}}</text><text class="nav-action" @tap="save">{{actionLabel}}</text></view>
    <view v-if="type==='search'" class="search-box"><input v-model="keyword" placeholder="搜索昵称、兴趣、城市" confirm-type="search"/><text @tap="search">搜索</text></view>
    <view v-if="type==='edit'" class="edit-wrap">
      <view class="edit-hero">
        <view class="avatar-edit" @tap="chooseAvatar">
          <image v-if="profile.avatar" :src="profile.avatar" mode="aspectFill"/>
          <text v-else>{{(profile.nickname||'遇').slice(0,1)}}</text>
          <view class="avatar-camera">＋</view>
        </view>
        <text class="avatar-hint">点击更换头像</text>
        <text class="edit-tip">完善资料，让有趣的人更容易认识你</text>
      </view>
      <view class="form-card">
        <view class="form-heading">基本资料</view>
        <view class="field">
          <view class="field-label">昵称 <text class="required">*</text></view>
          <view class="field-control"><input v-model="profile.nickname" maxlength="20" placeholder="给自己取个好听的名字" placeholder-class="field-placeholder"/></view>
          <view class="field-meta">{{(profile.nickname||'').length}}/20</view>
        </view>
        <view class="field">
          <view class="field-label">所在城市</view>
          <view class="field-control"><input v-model="profile.city" maxlength="20" placeholder="例如：杭州" placeholder-class="field-placeholder"/></view>
        </view>
        <view class="field field-area">
          <view class="field-label">个人介绍</view>
          <view class="field-control"><textarea v-model="profile.bio" maxlength="120" placeholder="介绍一下自己，让大家更了解你…" placeholder-class="field-placeholder" /></view>
          <view class="field-meta">{{(profile.bio||'').length}}/120</view>
        </view>
        <view class="field field-last">
          <view class="field-label">兴趣标签</view>
          <view class="field-control"><input v-model="profile.tags" maxlength="60" placeholder="用空格分隔，如：旅行 摄影 咖啡" placeholder-class="field-placeholder"/></view>
          <view class="field-note">添加感兴趣的内容，帮助你找到同好</view>
        </view>
      </view>
      <view class="form-footnote">你的资料仅用于展示个人信息，可随时修改</view>
      <button class="primary save-profile" @tap="save">保存资料</button>
    </view>
    <view v-else-if="type==='dynamic'" class="card">
      <textarea v-model="postText" placeholder="记录此刻的心情、生活和灵感..." maxlength="300"/>
      <view class="attach" @tap="choosePostImages">＋ 添加照片（{{postImages.length}}/9）</view><view v-if="postImages.length" class="image-preview"><image v-for="(src,i) in postImages" :key="src" :src="src" mode="aspectFill" @tap="removePostImage(i)"/></view>
      <button class="primary" @tap="publish">发布动态</button>
      <view class="section">我的动态</view>
      <view v-for="(p,i) in posts" :key="p.id||i" class="post"><text>{{p.text}}</text><view v-if="p.images?.length" class="post-images"><image v-for="src in p.images" :key="src" :src="src" mode="aspectFill" @tap="previewPostImage(src,p.images)"/></view><small>{{p.time}} · <text @tap="likePost(p)">♡ {{p.likes}}</text> · <text @tap="commentPost(p)">评论 {{p.comments?.length||0}}</text></small><view v-for="(cm,j) in p.comments" :key="j" class="comment">{{cm}}</view></view>
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
      <view v-else-if="type==='account'" class="account-card">
        <view class="account-summary"><view class="account-symbol">♙</view><view class="account-summary-text"><b>账号安全中心</b><small>管理登录方式与账号信息</small></view><view class="safe-mark">安全</view></view>
        <view class="account-section-title">登录信息</view>
        <view class="account-row" @tap="openAccountEdit('phone')"><view class="account-icon">▣</view><view class="account-main"><b>手机号</b><small>{{accountPhone||'尚未绑定手机号'}}</small></view><text class="account-action">{{accountPhone?'更换':'绑定'}} ›</text></view>
        <view class="account-row" @tap="openAccountEdit('password')"><view class="account-icon">⌑</view><view class="account-main"><b>登录密码</b><small>定期修改密码，保护账号安全</small></view><text class="account-action">修改 ›</text></view>
        <view class="account-section-title">账号管理</view>
        <view class="account-row"><view class="account-icon">✓</view><view class="account-main"><b>账号状态</b><small>当前账号状态正常</small></view><text class="account-state">正常</text></view>
        <view class="account-row" @tap="confirmDelete"><view class="account-icon danger-icon">!</view><view class="account-main"><b>注销账号</b><small>申请后将进入账号注销流程</small></view><text class="account-action">申请注销 ›</text></view>
        <view class="account-foot">请勿向他人透露验证码或登录密码</view>
      </view>
      <view v-if="accountModal" class="modal-mask" @tap="closeAccountEdit">
        <view class="edit-modal" @tap.stop>
          <view class="modal-grabber"></view><view class="modal-title">{{accountEditType==='phone'?'绑定 / 更换手机号':'修改登录密码'}}</view>
          <view class="modal-desc">{{accountEditType==='phone'?'请输入新的手机号。正式短信验证需接入服务端。':'设置一个安全且不易猜测的新密码'}}</view>
          <view v-if="accountEditType==='phone'" class="modal-field"><text>手机号</text><input v-model="accountInput" type="number" maxlength="11" placeholder="请输入11位手机号"/></view>
          <template v-else><view class="modal-field"><text>新密码</text><input v-model="accountInput" password placeholder="至少8位，包含字母和数字"/></view><view class="modal-field"><text>确认密码</text><input v-model="accountConfirm" password placeholder="再次输入新密码"/></view></template>
          <button class="modal-primary" @tap="submitAccountEdit">确认保存</button>
          <text class="modal-cancel" @tap="closeAccountEdit">暂不修改</text>
        </view>
      </view>
      <view v-else-if="type==='blacklist'" class="card"><view v-for="(person,i) in blocked" :key="person" class="person"><view class="avatar">{{person.slice(0,1)}}</view><view class="person-info"><b>{{person}}</b><small>已加入黑名单</small></view><text class="action" @tap="unblock(i)">移除</text></view><view v-if="!blocked.length" class="empty">黑名单为空，你可以安心认识新朋友</view></view>
      <view v-else-if="type==='about'||type==='agreement'" class="card legal"><view class="brand">遇见 MEET</view><b>{{type==='about'?'让每一次相遇都值得期待':'用户协议与隐私说明'}}</b><p>{{type==='about'?'遇见是一款帮助用户发现兴趣伙伴、分享生活和建立真实连接的社交产品。当前版本为可交互产品原型，账号、消息、音视频和直播服务尚未连接生产后端。':'请遵守法律法规与社区规范，尊重其他用户的意愿，不发布违法、骚扰、欺诈或侵犯他人权益的内容。个人资料和互动数据在当前演示版本中保存在本地设备。真实服务上线前需配置正式隐私政策、数据处理规则和用户申诉渠道。'}}</p><p>版本号：1.0.1</p></view>
      <view v-else-if="['likes','following','visitors','matches','channel','topic'].includes(type)" class="card">
        <view class="section">{{type==='likes'?'喜欢我的':type==='following'?'我的关注':type==='visitors'?'最近访客':type==='matches'?'今日匹配':type==='channel'?(routeName||'兴趣频道'):(routeName||'话题讨论')}}</view>
        <view v-for="(u,i) in collection" :key="u" class="person" @tap="openUser(u)"><view class="avatar">{{u.slice(0,1)}}</view><view class="person-info"><b>{{u}}</b><small>{{type==='visitors'?'刚刚看过你的主页':type==='matches'?'与你有共同兴趣 · 匹配度 '+(96-i*4)+'%':type==='channel'?'兴趣相投 · 活跃用户':'期待与你交流'}}</small></view><text class="action" @tap.stop="openUser(u)">查看 ›</text></view>
        <view v-if="type==='channel'||type==='topic'" class="topic-note">分享你的兴趣和生活，主动打个招呼，开启一段新的连接。</view>
      </view>
      <view v-else class="card"><view class="empty">暂无内容</view></view>
    </view>
    <view v-if="toast" class="toast" @tap="toast=''">{{toast}}</view>
  </view>
</template>
<script setup>
import {ref,computed} from 'vue'
import {onLoad} from '@dcloudio/uni-app'
const type=ref(''),title=ref('功能'),toast=ref(''),keyword=ref(''),postText=ref(''),postImages=ref([]),realName=ref(''),idNumber=ref(''),routeName=ref('')
const actionLabel=ref(''),accountModal=ref(false),accountEditType=ref('phone'),accountInput=ref(''),accountConfirm=ref(''),accountPhone=ref('')
const profile=ref({nickname:'遇见新朋友',city:'杭州',bio:'认真生活，慢慢遇见。',tags:'旅行 摄影 咖啡',avatar:''})
const allUsers=['浅安','小鱼','柒','江姜','早睡','优优','晚风','南栀','橘子海','林间','星野','桃桃']
const filteredUsers=computed(()=>allUsers.filter(x=>!keyword.value||x.includes(keyword.value)||['旅行','摄影','咖啡','音乐','电影','徒步','美食'].some(t=>t.includes(keyword.value))))
const collection=computed(()=>type.value==='likes'?['小鱼','浅安','柒']:type.value==='following'?['江姜','晚风','南栀']:type.value==='visitors'?['橘子海','星野','桃桃']:type.value==='matches'?['优优','林间','知夏']:type.value==='channel'?['小鱼','可乐','山茶','青禾']:['浅安','晚星','小鹿','云朵'])
const posts=ref([{id:1,text:'生活碎片：今天也要保持好奇心 ☀️',time:'今天 09:30',likes:23,liked:false,comments:['今天也要开心！']},{id:2,text:'周末的咖啡和一本好书。',time:'昨天 18:20',likes:16,liked:false,comments:[]}])
const blocked=ref(['测试用户A','测试用户B'])
const toggles=ref({online:true,dm:true,distance:true,likes:true,messages:true,system:true,vibration:false,sound:true})
const optionItems=computed(()=>type.value==='notifications'?[{key:'likes',title:'喜欢与关注',desc:'收到喜欢、关注时通知'},{key:'messages',title:'新消息提醒',desc:'收到私信时通知'},{key:'system',title:'系统通知',desc:'活动与安全提醒'}]:[{key:'sound',title:'提示音',desc:'播放消息提示音'},{key:'vibration',title:'振动反馈',desc:'收到消息时振动'}])
const titles={search:'搜索',edit:'编辑资料',dynamic:'我的动态',verify:'实名认证',privacy:'隐私设置',notifications:'消息通知',general:'通用设置',account:'账号与安全',blacklist:'黑名单',about:'关于我们',agreement:'用户协议',likes:'喜欢我的',following:'我的关注',visitors:'最近访客',matches:'今日匹配',channel:'兴趣频道',topic:'话题讨论'}
onLoad((query={})=>{
  type.value=query.type||'about'
  routeName.value=query.name||''
  title.value=titles[type.value]||routeName.value||'功能'
  actionLabel.value=type.value==='edit'?'保存':''
  try{
    accountPhone.value=uni.getStorageSync('meet-account-phone')||''
    const p=uni.getStorageSync('meet-profile')
    if(p)profile.value={...profile.value,...p}
    const t=uni.getStorageSync('meet-settings')
    if(t)toggles.value={...toggles.value,...t}
    const b=uni.getStorageSync('meet-blacklist')
    if(b)blocked.value=b
    const ps=uni.getStorageSync('meet-posts')
    if(ps)posts.value=ps
  }catch(e){}
})
function back(){uni.navigateBack()}
function toggle(k,v){toggles.value[k]=v;uni.setStorageSync('meet-settings',toggles.value)}
function save(){if(!profile.value.nickname?.trim()){toast.value='请先填写昵称';return}profile.value.nickname=profile.value.nickname.trim();profile.value.city=(profile.value.city||'').trim();profile.value.bio=(profile.value.bio||'').trim();profile.value.tags=(profile.value.tags||'').trim();uni.setStorageSync('meet-profile',profile.value);toast.value='资料已保存'}
function chooseAvatar(){uni.chooseImage({count:1,sizeType:['compressed'],success:r=>{profile.value.avatar=r.tempFilePaths[0];toast.value='头像已选择，记得保存资料'}})}
function choosePostImages(){const remain=9-postImages.value.length;if(remain<=0){toast.value='最多添加9张照片';return}uni.chooseImage({count:remain,sizeType:['compressed'],success:r=>{postImages.value=[...postImages.value,...r.tempFilePaths].slice(0,9)}})}
function removePostImage(i){postImages.value.splice(i,1)}
function previewPostImage(src,urls){uni.previewImage({current:src,urls})}
function publish(){if(!postText.value.trim()){toast.value='先写下你的动态内容';return}posts.value.unshift({id:Date.now(),text:postText.value.trim(),images:[...postImages.value],time:'刚刚',likes:0,liked:false,comments:[]});uni.setStorageSync('meet-posts',posts.value);postText.value='';postImages.value=[];toast.value='动态发布成功'}
function likePost(p){p.liked=!p.liked;p.likes=Math.max(0,p.likes+(p.liked?1:-1));uni.setStorageSync('meet-posts',posts.value)}
function commentPost(p){uni.showModal({title:'评论动态',editable:true,placeholderText:'写下你的评论',success:r=>{if(r.confirm&&r.content?.trim()){p.comments=p.comments||[];p.comments.push(r.content.trim());uni.setStorageSync('meet-posts',posts.value)}}})}
function verify(){if(!realName.value.trim()||idNumber.value.trim().length<8){toast.value='请填写姓名和有效证件号码';return}toast.value='资料格式已通过演示校验，正式认证需接入服务端'}
function search(){if(!keyword.value.trim())toast.value='请输入搜索关键词';else toast.value='已找到 '+filteredUsers.value.length+' 位相关用户'}
function openUser(u){uni.navigateTo({url:'/pages/user/index?name='+encodeURIComponent(u)})}
function unblock(i){blocked.value.splice(i,1);uni.setStorageSync('meet-blacklist',blocked.value);toast.value='已移出黑名单'}
function openAccountEdit(kind){accountEditType.value=kind;accountInput.value='';accountConfirm.value='';accountModal.value=true}
function closeAccountEdit(){accountModal.value=false}
function submitAccountEdit(){if(accountEditType.value==='phone'){if(!/^1[3-9][0-9]{9}$/.test(accountInput.value)){toast.value='请输入有效的11位手机号';return}accountPhone.value=accountInput.value;uni.setStorageSync('meet-account-phone',accountPhone.value);toast.value='手机号已保存（演示模式）'}else{if(!/^(?=.*[A-Za-z])(?=.*[0-9]).{8,}$/.test(accountInput.value)){toast.value='密码至少8位，且需包含字母和数字';return}if(accountInput.value!==accountConfirm.value){toast.value='两次输入的密码不一致';return}uni.setStorageSync('meet-account-password',accountInput.value);toast.value='密码已更新（本机演示）'}accountModal.value=false}
function confirmDelete(){uni.showModal({title:'申请注销账号',content:'注销会影响账号资料与本机数据。当前演示版本不会执行真实注销。',confirmText:'我知道了',showCancel:true})}
</script>
<style scoped lang="scss">
.page{min-height:100vh;background:linear-gradient(150deg,#e5efff,#f8e9f4 55%,#eef6ff);color:#342e40;padding:env(safe-area-inset-top) 16px 30px;box-sizing:border-box}.nav{height:52px;display:flex;align-items:center;gap:16px}.back{font-size:30px;color:#a47eb6}.nav-title{font-size:18px;font-weight:800;flex:1}.nav-action{font-size:12px;color:#c96da9}.card{background:#ffffffd9;border:1px solid #fff;border-radius:18px;padding:16px;margin-top:8px;box-shadow:0 5px 18px #a99ab512}.list{background:transparent;border:0;border-radius:0;padding:0;margin-top:8px;box-shadow:none}.search-box{display:flex;gap:10px;align-items:center;background:#fff;border-radius:16px;padding:8px 14px;margin:12px 0}.search-box input{flex:1}.search-box text,.action{color:#d46eaa}.edit-wrap{padding-bottom:18px}.edit-hero{padding:22px 16px 18px;text-align:center}.avatar-edit{position:relative;width:88px;height:88px;border-radius:50%;background:linear-gradient(135deg,#d8e7ff,#f6d8eb);margin:0 auto 9px;display:flex;align-items:center;justify-content:center;font-size:30px;font-weight:700;color:#9d79b2;box-shadow:0 7px 20px #a99ab52b}.avatar-edit image{width:88px;height:88px;border-radius:50%}.avatar-camera{position:absolute;right:0;bottom:0;width:25px;height:25px;border:2px solid #fff;border-radius:50%;background:#c77db4;color:#fff;font-size:18px;line-height:21px}.avatar-hint,.edit-tip{display:block}.avatar-hint{font-size:11px;color:#bd78ad}.edit-tip{font-size:11px;color:#a49aaa;margin-top:8px}.form-card{background:#fff;border:1px solid #fff;border-radius:20px;padding:5px 17px 3px;box-shadow:0 7px 24px #a99ab512}.form-heading{font-size:14px;font-weight:800;padding:15px 0 8px;color:#393244}.field{padding:13px 0 11px;border-bottom:1px solid #f1edf3}.field-last{border-bottom:0}.field-label{font-size:12px;font-weight:650;color:#62596c;margin-bottom:9px}.required{color:#e58ab2}.field-control{background:#f7f5f9;border:1px solid #f2eef5;border-radius:12px;padding:0 12px;min-height:46px;display:flex;align-items:center;box-sizing:border-box}.field-control input{height:44px;line-height:44px;padding:0;background:transparent;border:0;font-size:13px;color:#393244}.field-control textarea{width:100%;min-height:94px;padding:12px 0;background:transparent;border:0;font-size:13px;line-height:1.65;color:#393244;box-sizing:border-box}.field-area .field-control{align-items:flex-start}.field-placeholder{color:#b8b0bf;font-size:12px}.field-meta{text-align:right;color:#b6adbb;font-size:10px;margin-top:5px}.field-note{font-size:10px;color:#aaa1b0;margin-top:7px}.form-footnote{text-align:center;color:#aaa1b0;font-size:10px;margin:14px 0 4px}.save-profile{height:46px;line-height:46px;margin-top:16px;box-shadow:0 7px 16px #d98ab52b}.image-preview,.post-images{display:flex;flex-wrap:wrap;gap:7px;margin-top:10px}.image-preview image,.post-images image{width:calc(33.333% - 5px);height:96px;border-radius:10px}.post-images image{height:104px}label{display:block;font-size:12px;color:#8e8498;margin:11px 0 5px}input,textarea{width:100%;box-sizing:border-box;background:#f7f4fa;border-radius:12px;padding:12px;font-size:13px}textarea{min-height:110px}.primary{width:100%;margin-top:18px;border:0;border-radius:24px;background:linear-gradient(100deg,#f39bc4,#a99af0);color:white;font-size:14px}.section{font-weight:800;margin:14px 0 9px}.post{padding:14px 0;border-bottom:1px solid #f0ebf3;font-size:13px}.post small{display:block;color:#aaa;font-size:10px;margin-top:8px}.attach{padding:12px;background:#f7f3fa;border-radius:12px;color:#a77bb2;font-size:12px;margin-top:10px}.verify-icon{width:62px;height:62px;border-radius:50%;display:grid;place-items:center;background:#e5f6ed;color:#51b783;font-size:30px;margin:10px auto}.center-title{text-align:center;font-size:18px;font-weight:800}.muted{display:block;text-align:center;color:#a39aa9;font-size:11px;margin:10px 0 18px}.setting-row,.info-row{display:flex;align-items:center;justify-content:space-between;gap:12px;padding:12px 0;border-bottom:1px solid #f1edf4}.setting-row:last-child,.info-row:last-child{border:0}.setting-row b,.setting-row small{display:block}.setting-row b,.info-row{font-size:13px}.setting-row small{font-size:10px;color:#aaa;margin-top:5px}.info-row text{color:#8f8797}.info-row b{font-weight:500}.danger{color:#df7099}.person{display:flex;align-items:center;gap:12px;padding:12px 0;border-bottom:1px solid #f1edf4}.avatar{width:42px;height:42px;border-radius:50%;background:linear-gradient(130deg,#d3e4ff,#f5cce7);display:grid;place-items:center;color:#9879b1}.person-info{flex:1}.person-info b,.person-info small{display:block}.person-info small{font-size:10px;color:#aaa;margin-top:4px}.person>text{font-size:12px}.empty{text-align:center;color:#a49baa;padding:32px 10px;font-size:12px}.brand{font-size:21px;font-weight:900;color:#bd79b4;margin-bottom:20px}.legal b{font-size:15px}.legal p{font-size:12px;line-height:1.9;color:#777;margin-top:14px}.account-card{background:#fff;border:1px solid #fff;border-radius:20px;padding:16px 17px;box-shadow:0 8px 26px #a99ab51a}.account-summary{display:flex;align-items:center;padding:5px 0 19px;border-bottom:1px solid #f1edf3}.account-symbol{width:44px;height:44px;border-radius:14px;background:linear-gradient(135deg,#e4efff,#f8e2f0);display:grid;place-items:center;color:#a77bb7;font-size:23px;margin-right:12px}.account-summary-text{flex:1}.account-summary-text b,.account-summary-text small{display:block}.account-summary-text b{font-size:14px}.account-summary-text small{font-size:10px;color:#aaa;margin-top:5px}.safe-mark{font-size:10px;color:#55ad87;background:#eaf7f0;border-radius:20px;padding:5px 9px}.account-section-title{font-size:11px;font-weight:800;color:#9a91a2;margin:19px 0 6px}.account-row{display:flex;align-items:center;min-height:67px;border-bottom:1px solid #f4f1f5;gap:11px}.account-row:last-of-type{border-bottom:0}.account-icon{width:34px;height:34px;border-radius:11px;background:#f5f1f8;color:#a77bb7;display:grid;place-items:center;font-size:17px}.account-main{flex:1}.account-main b,.account-main small{display:block}.account-main b{font-size:13px;color:#393244}.account-main small{font-size:10px;color:#aaa;margin-top:5px}.account-action{font-size:11px;color:#bd78ad}.account-state{font-size:11px;color:#55ad87}.danger-icon{color:#d77a9b;background:#fff0f4}.account-foot{text-align:center;color:#b4acba;font-size:10px;padding:17px 0 2px}.modal-mask{position:fixed;z-index:20;inset:0;background:#211a2b66;display:flex;align-items:flex-end;justify-content:center;padding:0 12px env(safe-area-inset-bottom)}.edit-modal{width:100%;max-width:520px;background:#fff;border-radius:24px 24px 0 0;padding:12px 20px 24px;box-sizing:border-box;box-shadow:0 -10px 40px #33243b22}.modal-grabber{width:38px;height:4px;border-radius:5px;background:#e7e1e9;margin:0 auto 19px}.modal-title{font-size:17px;font-weight:800;text-align:center;color:#383143}.modal-desc{text-align:center;font-size:11px;color:#a49baa;margin:8px 0 19px;line-height:1.6}.modal-field{margin:12px 0}.modal-field>text{display:block;font-size:11px;color:#746b7d;margin-bottom:7px}.modal-field input{height:46px;background:#f7f5f9;border:1px solid #f0ecf3;border-radius:12px;padding:0 13px;font-size:13px;box-sizing:border-box}.modal-primary{height:46px;line-height:46px;border:0;border-radius:24px;background:linear-gradient(100deg,#f39bc4,#a99af0);color:white;font-size:14px;margin-top:18px}.modal-cancel{display:block;text-align:center;color:#a49baa;font-size:12px;padding:15px 0 2px}.toast{position:fixed;bottom:30px;left:10%;width:80%;text-align:center;background:#30283de8;color:white;padding:11px;border-radius:24px;font-size:12px}
</style>