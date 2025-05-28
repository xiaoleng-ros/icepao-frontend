<template>
  <div class="login-container">
    <h1 class="login-title">登录</h1>
    <div class="sun-icon">
      <img src="../assets/mao.png" alt="sun" />
    </div>
    <van-form @submit="onSubmit">
      <van-cell-group inset>
        <van-field
          v-model="userAccount"
          name="用户名"
          label="用户名"
          placeholder="用户名"
          :rules="[{ required: true, message: '请输入用户名' }]"
        />
        <van-field
          v-model="userPassword"
          type="password"
          name="密码"
          label="密码"
          placeholder="密码"
          :rules="[{ required: true, message: '请输入密码' }]"
        />
      </van-cell-group>
      <div class="button-group">
        <van-button round block type="primary" native-type="submit">
          提交
        </van-button>
      </div>
      <div class="link-group">
        <router-link to="/user/register" class="register-link">没有账号？去注册</router-link>
        <router-link to="/contact" class="forgot-password">忘记密码？联系站长</router-link>
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
const router = useRouter();
const route = useRoute();
const onSubmit = async () => {
  const res = await myAxios.post("/user/login", {
    userAccount: userAccount.value,
    userPassword: userPassword.value
  })
  console.log(res, "登录返回信息");
  if (res.code === 0 && res.data) {
    //跳转到之前页面
    const redirectUrl = route.query?.redirect as string ?? '/';
    showSuccessToast('登陆成功');
    window.location.href = redirectUrl;
  } else {
    showFailToast('登录失败'+res.description);
  }
};
</script>

<style scoped>
.login-container {
  padding: 20px;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.login-title {
  font-size: 24px;
  font-weight: bold;
  margin-bottom: 20px;
}

.sun-icon {
  width: 100px;
  height: 100px;
  margin: 20px 0;
}

.sun-icon img {
  width: 100%;
  height: 100%;
}

.button-group {
  margin: 16px;
  width: 100%;
}

.link-group {
  display: flex;
  justify-content: space-between;
  width: 100%;
  padding: 0 16px;
  margin-top: 16px;
}

.register-link,
.forgot-password {
  color: #1989fa;
  text-decoration: none;
  font-size: 14px;
}

:deep(.van-cell-group--inset) {
  margin: 0;
}

:deep(.van-field) {
  margin-bottom: 12px;
}
</style>
