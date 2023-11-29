<script setup>
import { ref, onMounted } from "vue";
import Card from "./Card.vue";
import { supabase } from "../supabase";
import { useUsersStore } from "../stores/users";
import { storeToRefs } from "pinia";
import Observer from "./Observer.vue";

const userStore = useUsersStore();
const { user } = storeToRefs(userStore);

const posts = ref([]);
const limitOfData = ref(2);
const ownersId = ref([]);
const reachEnd = ref(false);

const fetchData = async () => {
  const { data: followings } = await supabase
    .from("followers_following")
    .select("following_id")
    .eq("followers_id", user.value.id);

  ownersId.value = followings.map((user) => user.following_id);

  const { data } = await supabase
    .from("posts")
    .select()
    .in("owner_id", ownersId.value)
    .range(0, limitOfData.value)
    .order("created_at", { ascending: false });

  posts.value = data;
};

const fetchNextData = async () => {
  if (!reachEnd.value) {
    const { data } = await supabase
      .from("posts")
      .select()
      .in("owner_id", ownersId.value)
      .range(limitOfData.value + 1, limitOfData.value + 3)
      .order("created_at", { ascending: false });

    posts.value = [...posts.value, ...data];

    limitOfData.value = limitOfData.value + 3;

    if (!data.length) {
      reachEnd.value = true;
    }
  }
};

onMounted(() => {
  fetchData();
});
</script>
<template>
  <div class="card-container">
    <Card v-for="post in posts" :key="post.id" :post="post" />
    <Observer v-if="posts.length" @intersect="fetchNextData" />
  </div>
</template>
