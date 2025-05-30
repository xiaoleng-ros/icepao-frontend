<template>
  <van-form @submit="onSubmit">
    <van-field
        v-model="editUser.currentValue"
        :name="editUser.editKey"
        :label="editUser.editName"
        :placeholder="editUser.placeholder"
    />
    <div style="margin: 16px;">
      <van-button round block type="primary" native-type="submit">
        提交
      </van-button>
    </div>
  </van-form>
</template>

<script setup lang="ts">
import {useRoute, useRouter} from "vue-router";
import {ref} from "vue";
import {getCurrentUser} from "../services/user.js";
import {showFailToast, showSuccessToast} from "vant";
import myAxios from "../plugins/myAxios.js";

const router = useRouter();
const onSubmit = async () => {
  const currentUser =await getCurrentUser();
  if (!currentUser) {
    showFailToast('用户未登录');
    return;
  }
  console.log(currentUser,"用户信息");
  
  // 处理性别字段的特殊转换
  let submitValue = editUser.value.currentValue;
  if (editUser.value.editKey === 'gender') {
    if (submitValue === '男') {
      submitValue = 0;
    } else if (submitValue === '女') {
      submitValue = 1;
    } else {
      showFailToast('性别只能输入：男 或 女');
      return;
    }
  }
  
  const res = await myAxios.post('/user/update', {
    'id': currentUser.id,
    [editUser.value.editKey as string]: submitValue
  })
  console.log(res,"修改信息")
  if(res.code === 0 && res.data>0){
    showSuccessToast("修改成功");
  }else {
    showFailToast('修改失败'+res.description);
  }
};

const route = useRoute();
const editUser = ref({
  editKey: route.query.editKey,
  editName: route.query.editName,
  currentValue: route.query.currentValue,
  placeholder: route.query.placeholder,
})
</script>

<style scoped>

</style>
