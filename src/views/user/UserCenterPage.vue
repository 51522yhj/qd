<template>
  <a-card title="个人中心">
    <a-form :model="form" layout="vertical">
      <!-- 用户头像 -->

      <a-form-item label="用户头像">
        <img
          v-if="form.avatar"
          class="avatar-circle"
          :src="form.avatar"
          alt="用户头像"
        />

        <!-- <PictureUploader
          biz="user_avatar"
          :value="form.avatar"
          :onChange="(value) => (form.avatar = value)"
        /> -->

        <!-- <a-button style="margin-left: 10px" @click="updateAvatar"
          >修改头像</a-button
        > -->
        <a-upload action="/" :custom-request="updateAvatar" />
      </a-form-item>

      <!-- 昵称 -->
      <a-form-item label="昵称">
        <a-input v-model="form.nickname" placeholder="请输入昵称" />
      </a-form-item>

      <!-- <a-form-item label="账号">
        <a-input v-model="form.account" placeholder="请输入账号" />
      </a-form-item> -->
      <!-- 简介 -->
      <a-form-item label="简介">
        <a-textarea
          v-model="form.bio"
          placeholder="请输入简介"
          :auto-size="{ minRows: 3, maxRows: 5 }"
        />
      </a-form-item>

      <!-- 修改密码按钮 -->
      <a-form-item>
        <a-button type="primary" @click="handleUpdateProfile"
          >保存更改</a-button
        >
        <a-button
          style="margin-left: 10px"
          @click="showChangePasswordModal = true"
          >修改密码</a-button
        >
      </a-form-item>
    </a-form>

    <!-- 修改密码模态框 -->
    <a-modal
      v-model:visible="showChangePasswordModal"
      title="修改密码"
      @ok="handleChangePassword"
    >
      <a-form :model="passwordForm" layout="vertical">
        <a-form-item label="旧密码">
          <a-input-password
            v-model="passwordForm.oldPassword"
            placeholder="请输入旧密码"
          />
        </a-form-item>
        <a-form-item label="新密码">
          <a-input-password
            v-model="passwordForm.newPassword"
            placeholder="请输入新密码"
          />
        </a-form-item>
        <a-form-item label="确认新密码">
          <a-input-password
            v-model="passwordForm.confirmNewPassword"
            placeholder="请确认新密码"
          />
        </a-form-item>
      </a-form>
    </a-modal>
  </a-card>
</template>

<script setup lang="ts">
import { ref, reactive, onMounted } from "vue";
import { Message } from "@arco-design/web-vue";
import PictureUploader from "@/components/PictureUploader.vue"; // 假设这是你的图片上传组件路径
import { useLoginUserStore } from "@/store/userStore";
import {
  updateUserUsingPost,
  resetPasswordUsingPost,
} from "@/api/userController";
import { uploadFileUsingPost } from "@/api/fileController";
// 获取登录用户
const loginUserStore = useLoginUserStore();
// 用户信息表单
const form = reactive({
  id: 0,
  avatar: "",
  nickname: "",
  bio: "",
});

onMounted(() => {
  console.log("登录用户:", loginUserStore.loginUser);

  form.avatar = loginUserStore.loginUser.userAvatar;
  form.nickname = loginUserStore.loginUser.userName;
  form.bio = loginUserStore.loginUser.userProfile;

  console.log(form.value, "form.value---------");

  // form.account = loginUserStore.loginUser.;
});
// 初始化表单数据
// form.avatar = user.value.avatar;
// form.nickname = user.value.nickname;
// form.bio = user.value.bio;

// 修改密码模态框显示状态
const showChangePasswordModal = ref(false);

// 密码表单
const passwordForm = reactive({
  oldPassword: "",
  newPassword: "",
  confirmNewPassword: "",
});
//修改头像
const updateAvatar = async (option: any) => {
  const { onError, onSuccess, fileItem } = option;
  console.log(fileItem);
  const res: any = await uploadFileUsingPost(
    { biz: "user_avatar" },
    {},
    fileItem.file
  );
  // 这里可以上传头像的逻辑
  if (res.data.code === 0 && res.data.data) {
    const url = res.data.data;
    form.avatar = url;
  }
};

// 保存用户信息
const handleUpdateProfile = async () => {
  // 这里可以添加保存用户信息的逻辑
  console.log("更新用户信息:", form);
  const res = await updateUserUsingPost({
    id: form.id,
    userAvatar: form.avatar,
    userName: form.nickname,
    userProfile: form.bio,
  });
  if (res.data.data == true) {
    Message.success("更新成功");
    loginUserStore.loginUser.userAvatar = form.avatar;
    loginUserStore.loginUser.userName = form.nickname;
    loginUserStore.loginUser.userProfile = form.bio;
  } else {
    Message.error("更新失败");
  }
};

// 修改密码
const handleChangePassword = async () => {
  if (passwordForm.newPassword !== passwordForm.confirmNewPassword) {
    Message.error("新密码和确认新密码不一致");
    return;
  }
  // 这里可以添加修改密码的逻辑
  const res = await resetPasswordUsingPost({
    oldPassword: passwordForm.oldPassword,
    newPassword: passwordForm.newPassword,
  });
  console.log("修改密码:", passwordForm);
  showChangePasswordModal.value = false;
  Message.success("密码修改成功");
};
</script>

<style scoped>
.avatar-circle {
  width: 90px !important;
  height: 90px !important;
  overflow: hidden !important;
  border-radius: 50% !important;
}
.avatar-circle img {
  width: 100% !important;
  height: 100% !important;
  object-fit: cover !important;
}
</style>
