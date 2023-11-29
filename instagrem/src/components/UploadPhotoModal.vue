<script setup>
import { ref, defineProps } from "vue";
import { supabase } from "../supabase";
import { useUsersStore } from "../stores/users";
import { storeToRefs } from "pinia";

const userStore = useUsersStore();

const { user } = storeToRefs(userStore);
const loading = ref(false);
const errorMessgae = ref("");
const open = ref(false);
const props = defineProps(["addNewPost"]);
const caption = ref("");
const file = ref(null);
const showModal = () => {
  open.value = true;
};

const handleOk = async () => {
  loading.value = true;
  const fileName = Math.floor(Math.random() * 1000000000);
  if (file.value) {
    const { data, error } = await supabase.storage
      .from("images")
      .upload("public/" + fileName, file.value);

    if (error) {
      loading.value = false;
      errorMessgae.value = "Unable to upload image";
    }

    await supabase.from("posts").insert({
      url: data.path,
      caption: caption.value,
      owner_id: user.value.id,
    });

    props.addNewPost({
      url: data.path,
      caption: caption.value,
    });
  }
  loading.value = false;
  caption.value = "";
  open.value = false;
};

const handleUploadPhoto = (e) => {
  if (e.target.files[0]) {
    file.value = e.target.files[0];
  }
};
</script>
<template>
  <div>
    <a-button @click="showModal">Upload Photo</a-button>
    <a-modal v-model:open="open" title="Upload photo" @ok="handleOk">
      <div v-if="!loading">
        <input type="file" accept=".jpeg, .png" @change="handleUploadPhoto" />
        <a-input
          v-model:value="caption"
          placeholder="Caption.."
          :maxLength="50"
        />
      </div>
      <div v-else class="spinner">
        <a-spin />
      </div>

      <a-typography-text v-if="errorMessage" type="danger">{{
        errorMessage
      }}</a-typography-text>
    </a-modal>
  </div>
</template>

<style scoped>
input {
  margin-top: 5px;
}

.spinner {
  display: flex;
  align-items: center;
  justify-content: center;
}
</style>
