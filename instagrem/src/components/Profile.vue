<script setup>
import Container from "./Container.vue";
import ImageGallery from "./ImageGallery.vue";
import UserBar from "./UserBar.vue";
import { ref, onMounted, watch, reactive } from "vue";
import { supabase } from "../supabase";
import { useRoute } from "vue-router";
import { useUsersStore } from "../stores/users";
import { storeToRefs } from "pinia";

const posts = ref([]);
const route = useRoute();
const { username } = route.params;
const user = ref(null);
const loading = ref(false);
const isFollowing = ref(false);

const userStore = useUsersStore();
const { user: loggedInUser } = storeToRefs(userStore);

const userInfo = reactive({ posts: null, followers: null, following: null });

const addNewpost = (post) => {
  posts.value.unshift(post);
};

const fetchData = async () => {
  loading.value = true;
  const { data: userData } = await supabase
    .from("users")
    .select()
    .eq("username", username)
    .single();

  if (!userData) {
    loading.value = false;
    return (user.value = null);
  }

  user.value = userData;

  const { data: postData } = await supabase
    .from("posts")
    .select()
    .eq("owner_id", user.value.id);

  await fetchFollowing();

  const followersCount = await fetchFollowersCount();

  const followingCount = await fetchFollowingCount();
  posts.value = postData;
  loading.value = false;

  userInfo.followers = followersCount;
  userInfo.following = followingCount;
  userInfo.posts = posts.value.length;
};

const fetchFollowersCount = async () => {
  const { count } = await supabase
    .from("followers_following")
    .select("*", { count: "exact" })
    .eq("following_id", user.value.id);

  return count;
};

const fetchFollowingCount = async () => {
  const { count } = await supabase
    .from("followers_following")
    .select("*", { count: "exact" })
    .eq("followers_id", user.value.id);

  return count;
};

const updateisFollowing = (follow) => {
  isFollowing.value = follow;
};

const fetchFollowing = async () => {
  if (loggedInUser.value && loggedInUser.value.id !== user.value.id) {
    const { data } = await supabase
      .from("followers_following")
      .select()
      .eq("followers_id", loggedInUser.value.id)
      .eq("following_id", user.value.id)
      .single();

    if (data) isFollowing.value = true;
  }
};

watch(loggedInUser, () => {
  fetchFollowing();
});

onMounted(() => {
  fetchData();
});
</script>

<template>
  <Container>
    <div class="profile-container" v-if="!loading">
      <UserBar
        :key="$route.params.username"
        :addNewPost="addNewpost"
        :userInfo="userInfo"
        :user="user"
        :isFollowing="isFollowing"
        :updateisFollowing="updateisFollowing"
      />
      <ImageGallery :posts="posts" />
    </div>
    <div class="spinner" v-else>
      <a-spin />
    </div>
  </Container>
</template>

<style scoped>
.profile-container {
  display: flex;
  flex-direction: column;
  padding: 20px 0;
}
.spinner {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 120px;
}
</style>
