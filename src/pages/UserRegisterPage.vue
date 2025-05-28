<template>
  <div class="register-container">
    <h1 class="register-title">注册</h1>
    <div class="brain-icon">
      <img src="../assets/mao.png" alt="brain" />
    </div>
    <van-form @submit="onSubmit">
      <van-cell-group inset>
        <van-field
          v-model="userAccount"
          name="用户名"
          label="用户名"
          placeholder="请输入用户名"
          :rules="[{ required: true, message: '请输入用户名' }]"
        />
        <van-field
          v-model="userPassword"
          type="password"
          name="密码"
          label="密码"
          placeholder="请输入密码"
          :rules="[{ required: true, message: '请输入密码' }]"
        />
        <van-field
          v-model="checkPassword"
          type="password"
          name="确认密码"
          label="确认密码"
          placeholder="请再次输入密码"
          :rules="[{ required: true, validator, message: '请重新输入密码' }]"
        />
        <van-field
          v-model="inviteCode"
          name="邀请码"
          label="邀请码"
          placeholder="请输入邀请码（选填）"
        />
      </van-cell-group>
      <div class="button-group">
        <van-button round block type="primary" native-type="submit">
          注册
        </van-button>
        <van-button round block type="primary" to="/user/login" class="login-button">
          已有账号？去登录
        </van-button>
      </div>
    </van-form>
  </div>
</template>
<script setup lang="ts">
import {ref} from "vue";
import {useRoute, useRouter} from "vue-router";
import myAxios from "../plugins/myAxios";
import {showFailToast, showSuccessToast} from "vant";

const userAccount = ref('');
const userPassword = ref('');
const checkPassword = ref('');
const inviteCode = ref('');
const router = useRouter();
const route = useRoute();



const onSubmit = async () => {
  const res = await myAxios.post("/user/register", {
    userAccount: userAccount.value,
    userPassword: userPassword.value,
    checkPassword: checkPassword.value,
  })
  if (res.code === 0 && res.data) {
    //跳转到之前页面
    const redirectUrl = route.query?.redirect as string ?? '/';
    showSuccessToast('注册成功');
    window.location.href = redirectUrl;
  } else {
    showFailToast('注册失败 '+res.description);
  }
};
const validator = (val : string) =>{
  if (val === userPassword.value){
    return true;
  }
  return false;
}
</script>

<style scoped>
.register-container {
  padding: 20px;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.register-title {
  font-size: 24px;
  font-weight: bold;
  margin-bottom: 20px;
}

.brain-icon {
  width: 120px;
  height: 120px;
  margin: 20px 0;
}

.brain-icon img {
  width: 100%;
  height: 100%;
}

.button-group {
  margin: 16px;
  width: 100%;
}

.login-button {
  margin-top: 12px;
  background-color: #1989fa;
  border-color: #1989fa;
}

:deep(.van-cell-group--inset) {
  margin: 0;
}

:deep(.van-field) {
  margin-bottom: 12px;
}
</style>
