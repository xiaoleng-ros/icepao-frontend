<template>
  <van-cell center title="心动模式">
    <template #right-icon>
      <van-switch v-model="isMatchMode" />
    </template>
  </van-cell>
  <UserCardList :user-list="userList" :loading="loading"/>
  <template v-if="!isMatchMode">
    <van-empty v-if="!userList || userList.length < 1" description="无符合的用户"/>
    <van-sticky v-if="totalItems > 15" :offset-top="50" @click="loadDate()">
      <van-pagination
          v-model="currentPage"
          :total-items="totalItems"
          :show-page-size="3"
          force-ellipses
          style="margin-bottom: 0"
      />
    </van-sticky>
  </template>
</template>

<script setup lang="ts">
import UserCardList from "../components/UserCardList.vue";
import myAxios from "../plugins/myAxios";
import {ref, watchEffect} from "vue";
import {showFailToast, showSuccessToast} from "vant";

const userList = ref([]);
const isMatchMode = ref<boolean>(false);
const loading = ref<boolean>(true);
const currentPage = ref(1);
const totalItems = ref(0); // 添加动态总数

const loadDate = async () => {
  let userListData;
  loading.value = true;
  
  // 清空现有数据，防止显示旧数据
  userList.value = [];
  
  if (isMatchMode.value) {
    const num = 5;
    userListData = await myAxios.get('/user/match', {
      params: {
        num,
        _t: Date.now() // 添加时间戳防止缓存
      },
      headers: {
        'Cache-Control': 'no-cache',
        'Pragma': 'no-cache'
      }
    })
    .then(function (response) {
      console.log('心动模式API响应:', response);
      showSuccessToast("开启心动模式");
      const userData = response?.data;
      if (userData && Array.isArray(userData)) {
        // 过滤掉已删除的用户（userState !== 0 表示正常用户）
        const activeUsers = userData.filter(user => user.userState === 0);
        totalItems.value = activeUsers.length;
        activeUsers.forEach(user => {
          if (!user.avatarUrl) {
            user.avatarUrl = '../assets/mao.png';
          }
        });
        return activeUsers;
      }
      return userData;
    })
    .catch(function (error) {
      console.log(error);
      totalItems.value = 0;
    });
  } else {
    userListData = await myAxios.get('/user/recommend', {
      params: {
        pageNum: currentPage.value,
        pageSize: 15,
        _t: Date.now() // 添加时间戳防止缓存
      },
      headers: {
        'Cache-Control': 'no-cache',
        'Pragma': 'no-cache'
      }
    })
    .then(function (response) {
      console.log('推荐模式API响应:', response);
      if (response?.data === null) {
        showFailToast('' + (response.description ? `${response.description}` : ''));
        totalItems.value = 0;
        return null;
      }
      
      const userData = response?.data;
      if (userData && Array.isArray(userData)) {
        // 过滤掉已删除的用户（userState !== 0 表示正常用户）
        const activeUsers = userData.filter(user => user.userState === 0);
        // 如果后端返回了总数信息，使用后端的总数
        totalItems.value = response?.total || activeUsers.length;
        activeUsers.forEach(user => {
          if (!user.avatarUrl) {
            user.avatarUrl = '../assets/mao.png';
          }
        });
        return activeUsers;
      } else {
        totalItems.value = 0;
      }
      return userData;
    })
    .catch(function (error) {
      console.log(error);
      totalItems.value = 0;
    });
  }
  
  userList.value = userListData || [];
  loading.value = false;
  console.log("当前用户数量:", userList.value.length);
  console.log("总用户数:", totalItems.value);
};

watchEffect(() => {
  loadDate();
});
</script>

<style scoped>

</style>
