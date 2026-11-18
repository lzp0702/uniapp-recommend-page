<template>
<view class="page">
  <view class="glow"></view>
  <view class="brand">遇见 <text>MEET</text></view>
  <view class="hero"><text class="eyebrow">MAKE EVERY CONNECTION COUNT</text><text class="title">让相遇，刚刚好。</text><text class="sub">发现有趣的人，分享生活里的每个瞬间</text></view>
  <view class="panel">
    <view class="switch"><text :class="{on:mode==='login'}" @tap="changeMode('login')">登录</text><text :class="{on:mode==='register'}" @tap="changeMode('register')">注册</text></view>
    <view class="field"><text class="field-label">手机号</text><view class="input-wrap"><text class="prefix">+86</text><input v-model="phone" type="number" maxlength="11" placeholder="请输入11位手机号" placeholder-class="placeholder"/></view></view>
    <view class="field"><text class="field-label">{{loginType==='code'?'短信验证码':'登录密码'}}</text><view class="input-wrap"><input v-model="credential" :password="loginType==='password'" type="text" :maxlength="loginType==='code'?6:32" :placeholder="loginType==='code'?'请输入验证码':'请输入密码'" placeholder-class="placeholder"/><text v-if="loginType==='code'" class="code-btn" @tap="sendCode">{{countdown>0?countdown+'秒后重试':'获取验证码'}}</text></view></view>
    <view v-if="mode==='login'" class="form-links"><text @tap="loginType=loginType==='code'?'password':'code'">{{loginType==='code'?'使用密码登录':'使用验证码登录'}}</text><text @tap="changeMode('reset')">忘记密码？</text></view>
    <view v-if="mode==='register'" class="form-links register-note">注册即表示同意《用户协议》和《隐私政策》</view>
    <button class="primary" :disabled="loading" @tap="submit">{{loading?'请稍候…':mode==='register'?'创建账号':mode==='reset'?'重置密码':'登录'}}</button>
    <view class="demo-note">{{notice}}</view>
  </view>
  <text class="bottom">继续即代表你已年满18周岁</text>
  <view v-if="toast" class="toast" @tap="toast=''">{{toast}}</view>
</view>
</template>
<script setup>
import{ref}from'vue'
const mode=ref('login'),loginType=ref('code'),phone=ref(''),credential=ref(''),notice=ref(''),toast=ref(''),countdown=ref(0),loading=ref(false)
let countdownTimer=null
function changeMode(value){mode.value=value;credential.value='';notice.value='';if(value==='register')loginType.value='code'}
function sendCode(){if(!/^1[3-9][0-9]{9}$/.test(phone.value)){notice.value='请先输入正确的11位手机号';return}countdown.value=60;notice.value='演示验证码已生成：123456（未发送真实短信）';const tick=()=>{if(countdown.value>0){countdown.value--;countdownTimer=setTimeout(tick,1000)}};tick()}
function submit(){if(loading.value)return;if(!/^1[3-9][0-9]{9}$/.test(phone.value)){notice.value='请输入正确的11位中国大陆手机号';return}if(!credential.value.trim()){notice.value=loginType.value==='code'?'请输入验证码':'请输入密码';return}
  if(mode.value==='reset'){if(loginType.value==='code'&&credential.value!=='123456'){notice.value='验证码错误，演示验证码为123456';return}notice.value='验证码校验通过。正式重置密码需连接服务端';return}
  if(loginType.value==='code'&&credential.value.trim()!=='123456'){notice.value='验证码不正确。演示验证码为123456';return}
  if(loginType.value==='password'){const saved=uni.getStorageSync('meet-account-password');if(saved&&saved!==credential.value){notice.value='密码不正确，请检查后重试';return}if(!saved){notice.value='当前账号未设置密码，请切换验证码登录';return}}
  if(mode.value==='register'&&loginType.value==='password'&&credential.value.length<8){notice.value='密码至少需要8位';return}
  loading.value=true
  if(mode.value==='register'&&loginType.value==='password')uni.setStorageSync('meet-account-password',credential.value)
  uni.setStorageSync('meet-session',{phone:phone.value,loginAt:Date.now(),mode:mode.value})
  uni.setStorageSync('meet-account-phone',phone.value)
  notice.value='验证成功，正在进入…'
  setTimeout(()=>{uni.reLaunch({url:'/pages/index/index',fail:()=>{loading.value=false;notice.value='页面跳转失败，请重试';}})},250)
}
</script>
<style scoped lang="scss">
.page{min-height:100vh;padding:54px 24px 28px;box-sizing:border-box;color:#302b40;background:linear-gradient(150deg,#e0edff,#f8e5f3 55%,#e4f5ff);position:relative;overflow:hidden}.glow{position:absolute;width:280px;height:280px;border-radius:50%;background:#fff;filter:blur(80px);opacity:.7;right:-100px;top:80px}.brand{font-size:25px;font-weight:900;position:relative}.brand text{font-size:10px;letter-spacing:4px;color:#a69ac4;margin-left:8px}.hero{margin:72px 0 30px;position:relative}.eyebrow{font-size:9px;letter-spacing:2px;color:#a18cb8;display:block}.title{font-size:32px;font-weight:900;display:block;margin:14px 0}.sub{font-size:13px;color:#91899c}.panel{position:relative;background:#ffffffd9;border:1px solid #fff;border-radius:26px;padding:23px 19px;box-shadow:0 16px 50px #a89bbd30;backdrop-filter:blur(18px)}.switch{display:flex;gap:28px;margin-bottom:23px;font-weight:700;color:#aaa2b3}.switch text{padding-bottom:7px}.switch .on{color:#d66cae;font-size:20px;border-bottom:2px solid #d88abb}.field{margin:15px 0}.field-label{display:block;font-size:11px;color:#756c80;margin:0 0 8px 3px}.input-wrap{height:49px;border-radius:13px;background:#f7f4fa;border:1px solid #f0ebf4;display:flex;align-items:center;padding:0 13px;box-sizing:border-box}.prefix{font-size:13px;color:#62596d;border-right:1px solid #e5deea;padding-right:10px;margin-right:10px}.input-wrap input{height:46px;flex:1;min-width:0;font-size:13px;background:transparent}.placeholder{color:#b5adbc}.code-btn{font-size:11px;color:#c66da8;white-space:nowrap;padding-left:8px}.form-links{display:flex;justify-content:space-between;color:#b07bb4;font-size:11px;margin:14px 2px}.register-note{justify-content:flex-start;color:#a59baa}.primary{height:47px;line-height:47px;background:linear-gradient(100deg,#f39bc4,#a99af0);color:white;border:0;border-radius:26px;font-size:15px;margin-top:12px}.primary[disabled]{opacity:.65}.demo-note{font-size:10px;color:#a59baa;line-height:1.6;text-align:center;margin-top:13px;min-height:17px}.bottom{display:block;text-align:center;color:#a59baa;font-size:11px;margin-top:26px}.toast{position:fixed;bottom:28px;left:10%;width:80%;text-align:center;background:#30283de8;color:white;padding:11px;border-radius:22px;font-size:12px}
</style>