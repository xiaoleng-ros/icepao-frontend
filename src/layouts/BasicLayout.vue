<template>
  <van-nav-bar
      :title="title"
      right-text="按钮"
      left-arrow
      fixed
      placeholder
      @click-left="onClickLeft"
      @click-right="onClickRight"
  >
    <template #right>
      <van-icon name="search" size="18" />
    </template>
  </van-nav-bar>
  <div id="content">
    <router-view />
  </div>
  <van-tabbar route placeholder>
    <van-tabbar-item to="/" icon="home-o" name="index">主页</van-tabbar-item>
    <van-tabbar-item to="/team" icon="search" name="team">队伍</van-tabbar-item>
    <van-tabbar-item to="/post" icon="fire-o" name="post">贴吧</van-tabbar-item>
    <van-tabbar-item to="/user/chat" icon="chat-o" name="chat">聊天</van-tabbar-item>
    <van-tabbar-item to="/notice" icon="comment-o" name="notice" :badge="unreadCount > 0 ? unreadCount : ''">信息</van-tabbar-item>
    <van-tabbar-item to="/user" icon="friends-o" name="user">个人</van-tabbar-item>
  </van-tabbar>
</template>

<script setup>
  import {showToast} from "vant";
  import {useRouter} from "vue-router";
  import {ref, onMounted, watch} from "vue";
  import routes from "../config/router.ts";
  import myAxios from "../plugins/myAxios";
  import {getCurrentUser} from "../services/user";

  const router = useRouter();
  const DEFAULT_TITLE = "伙伴匹配";
  const title = ref(DEFAULT_TITLE);
  const unreadCount = ref(0);

  // 获取未读通知数量
  const getUnreadNoticeCount = async () => {
    try {
      const user = await getCurrentUser();
      if (!user) return;
      
      const res = await myAxios.get('/notice/list');
      if (res?.code === 0 && res?.data) {
        // 统计未读通知数量（noticeState === 0 表示未读）
        const unreadNotices = res.data.filter(notice => notice.noticeState === 0);
        unreadCount.value = unreadNotices.length;
      }
    } catch (error) {
      console.error('获取未读通知数量失败:', error);
    }
  };

  // 监听路由变化，更新标题和未读通知数量
  router.beforeEach((to, from) => {
    const toPath = to.path;
    const route = routes.find((route) => {
      return toPath == route.path;
    });
    title.value = route?.title ?? DEFAULT_TITLE;
    
    // 当进入通知页面时，刷新未读通知数量
    if (toPath === '/notice') {
      setTimeout(() => {
        getUnreadNoticeCount();
      }, 500); // 延迟刷新，确保通知页面的操作完成
    }
  });

  // 组件挂载时获取未读通知数量
  onMounted(() => {
    getUnreadNoticeCount();
    
    // 定期刷新未读通知数量（可选）
    setInterval(() => {
      getUnreadNoticeCount();
    }, 30000); // 每30秒刷新一次
  });

  const onClickLeft = () => {
    router.back();
  };

  const onClickRight = () => {
    router.push('/search');
  };
</script>

<style scoped>
#content {
  padding-bottom: 50px;
}
</style>
