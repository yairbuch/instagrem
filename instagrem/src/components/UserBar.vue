<script setup>
import { defineProps } from "vue";
import UploadPhotoModal from "./UploadPhotoModal.vue";
import { useUsersStore } from "../stores/users";
import { storeToRefs } from "pinia";
import { useRoute } from "vue-router";
import { supabase } from "../supabase";

const route = useRoute();

const { username: profileUsername } = route.params;

const props = defineProps([
  "user",
  "userInfo",
  "addNewPost",
  "isFollowing",
  "updateisFollowing",
]);

const userStore = useUsersStore();

const { user } = storeToRefs(userStore);

const followUser = async () => {
  props.updateisFollowing(true);
  await supabase
    .from("followers_following")
    .insert({ followers_id: user.value.id, following_id: props.user.id });
  props.userInfo.followers += 1;
};

const unfollowUser = async () => {
  props.updateisFollowing(false);
  await supabase
    .from("followers_following")
    .delete()
    .eq("followers_id", user.value.id)
    .eq("following_id", props.user.id);
  props.userInfo.followers -= 1;
};
</script>
<template>
  <div class="userbar-container" v-if="props.user">
    <div class="top-content">
      <a-typography-title :level="2">{{
        props.user.username
      }}</a-typography-title>
      <div v-if="user">
        <UploadPhotoModal
          :addNewPost="addNewPost"
          v-if="user.username === profileUsername"
        />
        <div v-else>
          <AButton v-if="!props.isFollowing" @click="followUser"
            >Follow</AButton
          >
          <AButton v-else @click="unfollowUser">Following</AButton>
        </div>
      </div>
    </div>
    <div class="bottom-content">
      <a-typography-title :level="5"
        >{{ props.userInfo.posts }} posts</a-typography-title
      >
      <a-typography-title :level="5"
        >{{ props.userInfo.followers }} followers</a-typography-title
      >
      <a-typography-title :level="5">
        {{ props.userInfo.following }} following</a-typography-title
      >
    </div>
  </div>
  <div class="userbar-container" v-else>
    <div class="top-content">
      <a-typography-title :level="2">User not found</a-typography-title>
    </div>
  </div>
</template>

<style scoped>
.userbar-container {
  padding-bottom: 75px;
}
.bottom-content {
  display: flex;
  align-items: center;
}

.bottom-content h5 {
  margin: 0 !important;
  padding: 0;
  margin-right: 30px !important;
  align-items: center;
}

.top-content {
  display: flex;
  align-items: center;
  justify-content: space-between;
}
</style>
