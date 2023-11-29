<script setup>
import { ref, reactive } from "vue";
import { useUsersStore } from "../stores/users";
import { storeToRefs } from "pinia";

const props = defineProps(["isLogin"]);

const userStore = useUsersStore();
const open = ref(false);

const { errorMessage, loading, user } = storeToRefs(userStore);

const userCredentials = reactive({ username: "", password: "", email: "" });

const showModal = () => {
  open.value = true;
};

const clearInputs = () => {
  userCredentials.username = "";
  userCredentials.password = "";
  userCredentials.email = "";
  userStore.clearErrorMessage();
};

const handleOk = async (e) => {
  if (props.isLogin) {
    await userStore.handleLogin({
      email: userCredentials.email,
      password: userCredentials.password,
    });
  } else {
    await userStore.handleSignup(userCredentials);
  }

  if (user.value) {
    clearInputs();
    open.value = false;
  }
};

const handleCancel = () => {
  open.value = false;
  clearInputs();
};

const title = props.isLogin ? "Login" : "Sign up";
</script>
<template>
  <div>
    <a-button type="primary" @click="showModal" class="btn">{{
      title
    }}</a-button>

    <a-modal v-model:open="open" :title="title" @ok="handleOk">
      <template #footer>
        <a-button key="back" @click="handleCancel">Cancel</a-button>
        <a-button
          key="submit"
          type="primary"
          :loading="loading"
          @click="handleOk"
          :disabled="loading"
          >Submit</a-button
        >
      </template>
      <div v-if="!loading" class="input-container">
        <a-input
          class="input"
          v-if="!props.isLogin"
          v-model:value="userCredentials.username"
          placeholder="Username"
        />
        <a-input
          class="input"
          v-model:value="userCredentials.email"
          placeholder="Email"
        />
        <a-input
          class="input"
          v-model:value="userCredentials.password"
          placeholder="Password"
          type="password"
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
.btn {
  margin-left: 10px;
}
.input {
  margin-bottom: 5px;
}

.input-container {
  height: 120px;
}
.spinner {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 120px;
}
</style>
