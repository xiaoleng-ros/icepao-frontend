<template>
  <div
      id="postCardList"
      v-for="(post,index) in postList"
  >
    <van-cell-group inset>
      <van-card
          :title="post.username"
          :thumb="post.avatarUrl"
      >
        <template #desc >
          <div style="margin-top: 2px;color: darkgray" @click="doPostInfo(post.id)" >
            {{ post.content }}
          </div>
        </template>
        <template #footer>
          <van-divider/>
          <!-- 修改评论按钮为显示评论数量，点击跳转到详情页 -->
          <van-button size="small" @click="doPostInfo(post.id)" icon="chat-o">
            {{ post.commentCount || 0 }}
          </van-button>
          <van-button size="small" icon="good-job-o" @click="doThumb(index,post.id)" v-if="post.thumb" color="#ee0a24">{{post.thumbNum}}</van-button>
          <van-button size="small" icon="good-job-o" @click="doThumb(index,post.id)" v-if="!post.thumb">{{post.thumbNum}}</van-button>
          <van-button size="small" type="danger" v-if="post.userId === currentUser?.id" @click="doDeletePost(post.id)">
            删除
          </van-button>
        </template>
      </van-card>
      <!-- 移除评论列表显示 -->
    </van-cell-group>
    <van-divider :style="{ color: '#1989fa', borderColor: '#1989fa', padding: '0 16px' }"/>
  </div>
  <van-empty v-if="!postList || postList.length<1" description="无符合的帖子"/>
  
  <!-- 加载更多提示 -->
  <van-loading v-if="loading" type="spinner" color="#1989fa" style="text-align: center; padding: 20px;">
    加载中...
  </van-loading>
  
  <!-- 没有更多数据提示 -->
  <div v-if="finished && postList && postList.length > 0" style="text-align: center; padding: 20px; color: #999;">
    没有更多帖子了
  </div>
  
  <!-- 移除评论弹窗 -->
  <van-button icon="plus" type="primary" class="add-button" @click="doPostAdd"/>

</template>

<script setup lang="ts">
import {onMounted, ref, onUnmounted} from "vue";
import myAxios from "../plugins/myAxios";
import {showFailToast, showSuccessToast} from "vant";
import {postType} from "../models/post";
import {useRouter} from "vue-router";
import {getCurrentUser} from "../services/user";

const currentUser = ref('');
const postList = ref([]);
const router = useRouter();

// 无限滚动相关状态
const currentPage = ref(1);
const loading = ref(false);
const finished = ref(false);
const pageSize = 10; // 每页加载的帖子数量

interface postListProps {
  postList: postType[];
}

onMounted(async () => {
  await listPost();
  currentUser.value = await getCurrentUser();
  // 添加滚动监听
  window.addEventListener('scroll', handleScroll);
})

// 组件卸载时移除滚动监听
onUnmounted(() => {
  window.removeEventListener('scroll', handleScroll);
})

/**
 * 处理滚动事件
 */
const handleScroll = () => {
  // 获取滚动位置
  const scrollTop = window.pageYOffset || document.documentElement.scrollTop || document.body.scrollTop;
  // 获取文档总高度
  const scrollHeight = document.documentElement.scrollHeight || document.body.scrollHeight;
  // 获取可视窗口高度
  const clientHeight = window.innerHeight || document.documentElement.clientHeight || document.body.clientHeight;
  
  // 当滚动到距离底部100px时开始加载
  if (scrollTop + clientHeight >= scrollHeight - 100 && !loading.value && !finished.value) {
    loadMorePosts();
  }
};

/**
 * 加载更多帖子
 */
const loadMorePosts = async () => {
  if (loading.value || finished.value) return;
  
  loading.value = true;
  currentPage.value += 1;
  
  try {
    const res = await myAxios.get("/post/list", {
      params: {
        pageNum: currentPage.value,
        pageSize: pageSize,
      }
    });
    
    if (res && res.data && res.data.length > 0) {
      // 将新数据追加到现有列表
      postList.value = [...postList.value, ...res.data];
      
      // 如果返回的数据少于pageSize，说明没有更多数据了
      if (res.data.length < pageSize) {
        finished.value = true;
      }
    } else {
      // 没有更多数据
      finished.value = true;
    }
  } catch (error) {
    showFailToast("加载更多帖子失败");
    currentPage.value -= 1; // 回退页码
  } finally {
    loading.value = false;
  }
};

/**
 * 删除帖子
 */
const doDeletePost = async (id: number) => {
  const res = await myAxios.post("/post/delete", {
    id: id,
  })
  if (res?.code === 0 && res) {
    showSuccessToast("删除成功")
    // 删除成功后重新加载第一页数据
    await refreshPosts();
  } else {
    showFailToast('删除失败' + (res.description ? `,${res.description}` : ''))
  }
}

/**
 * 刷新帖子列表（重新从第一页开始加载）
 */
const refreshPosts = async () => {
  currentPage.value = 1;
  finished.value = false;
  postList.value = [];
  await listPost();
};

/**
 * 获取帖子的列表（初始加载）
 */
const listPost = async () => {
  loading.value = true;
  try {
    const res = await myAxios.get("/post/list", {
      params: {
        pageNum: 1,
        pageSize: pageSize,
      }
    });
    if (res && res.data) {
      postList.value = res.data;
      // 如果第一页数据就少于pageSize，说明没有更多数据
      if (res.data.length < pageSize) {
        finished.value = true;
      }
    } else {
      showFailToast("加载帖子失败")
    }
  } catch (error) {
    showFailToast("加载帖子失败")
  } finally {
    loading.value = false;
  }
};

/**
 * 添加帖子
 */
const doPostAdd = () => {
  router.push({
    path: "/post/add",
  })
}

/**
 * 点赞
 */
const doThumb = async (index:number,postId: number) => {
  const res = await myAxios.post("/post_thumb/", {
    postId: postId,
  })
  if (res?.code === 0 && res) {
    //返回为1表示点赞 返回为-1表示取消点赞
    if (res.data === 1){
      postList.value[index].thumbNum +=1;
    }else {
      postList.value[index].thumbNum -=1;
    }
    postList.value[index].thumb = !postList.value[index].thumb;
  } else {
    showFailToast('点赞失败' + (res.description ? `,${res.description}` : ''))
  }
}

/**
 * 跳转帖子详情
 */
const doPostInfo = (id: number) => {
  router.push({
    path: '/post/info',
    query: {
      id,
    }
  })
}
</script>

<style scoped>
.add-button {
  position: fixed;
  bottom: 80px;
  right: 20px;
  z-index: 999;
}
</style>
