<script setup>
import { RouterLink, useRouter, useRoute } from "vue-router";
import Container from "./Container.vue";
import { ref } from "vue";
import AuthModel from "./AuthModal.vue";
import { useUsersStore } from "../stores/users";
import { storeToRefs } from "pinia";

const route = useRoute();
const { username: profileUsername } = route.params;

const userStore = useUsersStore();

const { user, loadingUser } = storeToRefs(userStore);

const router = useRouter();

const searchUserName = ref("");

const onSearch = () => {
  if (searchUserName.value) {
    if (profileUsername) {
      router.push(`${searchUserName.value}`);
    } else {
      router.push(`/profile/${searchUserName.value}`);
    }
    searchUserName.value = "";
  }
};

const handleSignout = async () => {
  await userStore.handleLogout();
};

const goToUsersProfile = () => {
  if (profileUsername) {
    router.push(`${user.value.username}`);
  }
  router.push(`/profile/${user.value.username}`);
};
</script>
<template>
  <a-layout-header>
    <a-menu theme="dark" mode="horizontal" :style="{ lineHeight: '64px' }">
      <Container>
        <div class="nav-container">
          <div class="left">
            <a-menu-item key="1" style="color: rgb(64, 150, 255)">
              <RouterLink to="/">instagram</RouterLink>
            </a-menu-item>
            <a-input-search
              v-model:value="searchUserName"
              placeholder="user name..."
              style="width: 200px"
              @search="onSearch"
            />
          </div>
          <div class="content" v-if="!loadingUser">
            <div v-if="!user" class="right">
              <AuthModel :isLogin="false" />
              <AuthModel :isLogin="true" />
            </div>
            <div v-else class="right">
              <a-button type="primary" key="1" @click="goToUsersProfile">
                Profile
              </a-button>
              <a-button type="primary" key="2" @click="handleSignout"
                >Logout
              </a-button>
            </div>
          </div>
        </div>
      </Container>
    </a-menu>
  </a-layout-header>
</template>
<style scoped>
.nav-container {
  display: flex;
  justify-content: space-between;
}
.left {
  display: flex;
  align-items: center;
}
.right {
  margin-right: 10px;
  display: flex;
  align-items: center;
}
.right button {
  margin-left: 5px;
}

.content {
  display: flex;
  align-items: center;
}
</style>
