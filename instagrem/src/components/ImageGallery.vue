<script setup>
const props = defineProps(["posts"]);
const { VITE_BASE_PHOTO_URL } = import.meta.env;
</script>
<template>
  <div class="container-image-gallery">
    <img
      v-for="post in props.posts"
      :src="`${VITE_BASE_PHOTO_URL}${post.url}`"
      :key="post.id"
      alt=""
    />
  </div>
</template>

<style scoped>
.container-image-gallery {
  display: flex;
  justify-content: left;
  flex-wrap: wrap;
}
img {
  margin: 5px;
  width: 220px;
  height: 150px;
}
</style>
