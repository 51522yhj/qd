<template>
  <a-row id="globalHeader" align="center" :wrap="false">
    <a-col flex="auto">
      <a-menu
        mode="horizontal"
        :selected-keys="selectedKeys"
        @menu-item-click="doMenuClick"
      >
        <a-menu-item
          key="0"
          :style="{ padding: 0, marginRight: '38px' }"
          disabled
        >
          <div class="titleBar">
            <img class="logo" src="../assets/logo.png" />
            <div class="title">AI答题应用平台</div>
          </div>
        </a-menu-item>
        <a-menu-item v-for="item in visibleRoutes" :key="item.path">
          {{ item.name }}
        </a-menu-item>
      </a-menu>
    </a-col>
    <a-col flex="100px">
      <div v-if="loginUserStore.loginUser.id">
        <a-dropdown :trigger="['click']">
          <div class="avatar-dropdown">
            <a-avatar
              :image-url="
                loginUserStore.loginUser.userAvatar ??
                require('@/assets/user.png')
              "
            />
            <span
              class="user-name"
              style="width: 100px; text-align: center; display: inline-block"
            >
              {{ loginUserStore.loginUser.userName ?? "无名" }}
            </span>
          </div>
          <icon-down />

          <template #content>
            <a-doption @click="handleProfileClick">个人中心</a-doption>
            <a-doption @click="handleLogoutClick">退出登录</a-doption>
          </template>
        </a-dropdown>
      </div>
      <div v-else>
        <a-button type="primary" href="/user/login">登录</a-button>
      </div>

      <!-- <div>
        <a-button type="primary" status="danger" href="/user/login">logout</a-button>
      </div> -->
    </a-col>
  </a-row>
</template>

<script setup lang="ts">
import { routes } from "@/router/routes";
import { useRouter } from "vue-router";
import { computed, ref } from "vue";
import { useLoginUserStore } from "@/store/userStore";
import checkAccess from "@/access/checkAccess";
const handleProfileClick = () => {
  // 处理个人界面逻辑
  console.log("跳转到个人界面");
  router.push({
    path: "/usr/userCenter",
  });
};

const handleLogoutClick = () => {
  // 处理退出登录逻辑
  console.log("退出登录");
  router.push({
    path: "/user/login",
  });
};
const loginUserStore = useLoginUserStore();

const router = useRouter();
// 当前选中的菜单项
const selectedKeys = ref(["/"]);
// 路由跳转时，自动更新选中的菜单项
router.afterEach((to, from, failure) => {
  selectedKeys.value = [to.path];
});

// 展示在菜单栏的路由数组
const visibleRoutes = computed(() => {
  return routes.filter((item) => {
    if (item.meta?.hideInMenu) {
      return false;
    }
    // 根据权限过滤菜单
    if (!checkAccess(loginUserStore.loginUser, item.meta?.access as string)) {
      return false;
    }
    return true;
  });
});

// 点击菜单跳转到对应页面
const doMenuClick = (key: string) => {
  router.push({
    path: key,
  });
};
</script>

<style scoped>
#globalHeader {
}
.avatar-dropdown {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-right: 120px;
}

.user-name {
  margin-top: 4px; /* 根据需要调整间距 */
}
.arco-dropdown-link {
  color: #1677ff;
  cursor: pointer;
}
.titleBar {
  display: flex;
  align-items: center;
}

.title {
  margin-left: 16px;
  color: black;
}

.logo {
  height: 48px;
}
</style>
