<script setup>
import Likes from "./Likes.vue";
const { post } = defineProps(["post"]);
const { VITE_BASE_PHOTO_URL } = import.meta.env;
</script>
<template>
  <a-card hoverable style="width: 240px" class="card-container">
    <template #cover>
      <img alt="example" :src="`${VITE_BASE_PHOTO_URL}${post.url}`" />
    </template>
    <a-card-meta :title="post.username">
      <template #description>{{ post.caption }}</template>
    </a-card-meta>
    <Likes :post="post" />
  </a-card>
</template>

<style scoped>
.card-container {
  margin-top: 40px;
  padding: 5px;
}

img {
  width: 200px;
}
</style>
