<script setup>
import dayjs from "dayjs";
import { ref, computed } from "vue";
import relativeTime from "dayjs/plugin/relativeTime";
import {
  LikeFilled,
  LikeOutlined,
  DislikeFilled,
  DislikeOutlined,
} from "@ant-design/icons-vue";
import { useUsersStore } from "../stores/users";
import { storeToRefs } from "pinia";
import { supabase } from "../supabase";

const userStore = useUsersStore();

const { user } = storeToRefs(userStore);

const { post } = defineProps(["likes", "post"]);
dayjs.extend(relativeTime);

const action = ref();
const isLiked = ref(false);
let numOfLikes = ref(post.likes.length);

const like = async () => {
  if (
    isLiked.value ||
    (post.likes.includes(user.value.id) && action.value !== "disliked")
  ) {
    return alert("already liked");
  } else {
    action.value = "liked";
    const likesArray = post.likes.map((like) => like);
    const { data, error } = await supabase
      .from("posts")
      .update({ likes: [...likesArray, user.value.id] })
      .eq("id", post.id);
    isLiked.value = true;
    numOfLikes.value += 1;

    if (error) {
      console.log(error);
    }

    const dislikesArray = post.dislikes.filter(
      (dislike) => dislike !== user.value.id
    );

    const { errorInDislikes } = await supabase
      .from("posts")
      .update({ dislikes: [...dislikesArray] })
      .eq("id", post.id);
    if (errorInDislikes) {
      console.log(errorInDislikes);
    }
  }
};

const dislike = async () => {
  const { data: initialData } = await supabase
    .from("posts")
    .select("")
    .eq("id", post.id);

  if (initialData[0].dislikes.includes(user.value.id)) {
    return alert("already disliked");
  } else {
    action.value = "disliked";

    const likesArray = post.likes.filter((like) => like !== user.value.id);
    const { data, error } = await supabase
      .from("posts")
      .update({ likes: [...likesArray] })
      .eq("id", post.id);
    isLiked.value = false;

    if (error) {
      console.log(error);
    }

    const dislikesArray = post.dislikes.map((dislike) => dislike);
    const { error: errors } = await supabase
      .from("posts")
      .update({ dislikes: [...dislikesArray, user.value.id] })
      .eq("id", post.id);

    if (errors) {
      console.log(errors);
    }
    if (initialData[0].likes.includes(user.value.id)) {
      numOfLikes.value -= 1;
    }
  }
};

const shouldShowFilledLikeButton = computed(() => {
  return (
    action.value === "liked" ||
    (post.likes.includes(user.value.id) && action.value !== "disliked")
  );
});

const shouldShowFilledDisLikeButton = computed(() => {
  return (
    action.value === "disliked" ||
    (post.dislikes.includes(user.value.id) && action.value !== "liked")
  );
});
</script>
<template>
  <a-comment>
    <template #actions>
      <span key="comment-basic-like">
        <a-tooltip title="Like">
          <template v-if="shouldShowFilledLikeButton">
            <LikeFilled @click="like" />
          </template>
          <template v-else>
            <LikeOutlined @click="like" />
          </template>
        </a-tooltip>
        <span style="padding-left: 8px; cursor: auto">
          {{ numOfLikes }}
        </span>
      </span>
      <span key="comment-basic-dislike">
        <a-tooltip title="Dislike">
          <template v-if="shouldShowFilledDisLikeButton">
            <DislikeFilled @click="dislike" />
          </template>
          <template v-else>
            <DislikeOutlined @click="dislike" />
          </template>
        </a-tooltip>
        <span style="padding-left: 8px; cursor: auto"> </span>
      </span>
      <span key="comment-basic-reply-to">Reply to</span>
    </template>
  </a-comment>
</template>
