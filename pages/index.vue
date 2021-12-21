<template>
  <v-row>
    <v-alert shaped v-show="isShownAlert" type="warning" class="alert-box">
      Please fill up the form!
    </v-alert>
    <v-col cols="12" class="text-center">
      <h1>Join Chatroom</h1>
    </v-col>
    <v-col cols="12" class="text-center">
      <v-text-field
        solo
        clearable
        @keyup="validateForm"
        v-model="username"
        background-color="accent"
        label="Username"
        :rules="[
          (v) => !!v || 'Username is required',
          (v) => !v ? '' : (!v.match(/\s/g) || 'Username cannot contain spaces'),
        ]"
      />
    </v-col>
    <v-col cols="12" class="text-center">
      <v-text-field
        solo
        @keyup="validateForm"
        v-model="room"
        type="number"
        background-color="accent"
        label="Room"
        :rules="[(v) => !!v || 'Room is required']"
      />
    </v-col>
    <v-col cols="12" class="text-center">
      <v-btn
        block
        rounded
        color="primary"
        @click="joinRoom"
        :disabled="isJoinDisabled"
        >Join</v-btn
      >
    </v-col>
  </v-row>
</template>

<script>
export default {
  name: "IndexPage",
  data() {
    return {
      username: "",
      room: "",
      isShownAlert: false,
      isJoinDisabled: true,
    };
  },
  methods: {
    joinRoom() {
      if (!this.username.length || !this.room.length) {
        setTimeout(() => {
          this.isShownAlert = true;
        }, 3000);
        return;
      }
      localStorage.setItem("username", this.username.trim());
      location.href = "/room/" + this.room;
    },
    validateForm() {
      if (
        this.username.match(/\s/g) ||
        !this.username.length ||
        !this.room.length
      ) {
        this.isJoinDisabled = true;
      } else {
        this.isJoinDisabled = false;
      }
    },
  },
  mounted() {
    const username = localStorage.getItem("username");
    if (username != null) {
      this.username = username;
    }
  },
};
</script>

<style lang="sass" scoped>
.alert-box
  position: fixed
  bottom: 3vh
  right: 3vh
</style>
