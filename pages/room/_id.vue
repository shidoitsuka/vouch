<template>
  <v-row>
    <v-col cols="12" class="d-flex align-center">
      <span class="absolute primary--text"
        ><NuxtLink class="text-decoration-none" to="/">Exit</NuxtLink></span
      >
      <h1 class="mx-auto">Room #{{ $route.params.id }}</h1>
    </v-col>

    <v-col cols="12">
      <div class="chat-container" ref="chatContainer">
        <v-col
          cols="12"
          v-for="(chatItem, index) of chats"
          :key="chatItem._id"
          v-bind:class="{ 'text-right': chatItem.username == currentUser }"
          v-bind:ref="index == chats.length - 1 ? 'lastChat' : null"
        >
          <p v-if="chatItem.username != currentUser" class="username">
            {{ chatItem.username }}
          </p>
          <div
            class="triangle-chat rounded-chat"
            v-bind:class="{
              'chat-left': chatItem.username != currentUser,
              'chat-right': chatItem.username == currentUser,
            }"
          >
            <p class="chat-text">{{ chatItem.text }}</p>
          </div>
        </v-col>
      </div>
    </v-col>

    <v-col cols="12">
      <v-text-field
        hide-details
        solo
        rounded
        autofocus
        v-model="chat"
        background-color="accent"
        placeholder="Message here..."
        append-icon="mdi-arrow-up"
        @keyup.enter="sendMessage"
        @click:append="sendMessage"
      />
    </v-col>
  </v-row>
</template>

<script>
export default {
  name: "IndexPage",
  data() {
    return {
      chat: "",
      currentUser: "",
      tmpChats: [],
    };
  },
  computed: {
    chats() {
      return this.tmpChats;
    },
    chatHeight() {
      return this.$refs.chatContainer.clientHeight;
    }
  },
  beforeDestroy() {
    this.socket.disconnect();
  },
  mounted() {
    this.socket = this.$nuxtSocket({ channel: "/" });
    this.socket.on(`chat/${this.$route.params.id}`, (response) => {
      this.tmpChats.push(response);
    });
    this.currentUser = localStorage.getItem("username");
    this.getChatData();
  },
  updated() {
    this.scrollToBottomChat();
  },
  methods: {
    scrollToBottomChat() {
      this.$refs.chatContainer.scrollTop = this.chatHeight * this.chats.length;
    },
    sendMessage() {
      this.socket.emit("chat", { username: this.currentUser, room: this.$route.params.id, text: this.chat }, (res) => {});
      this.$axios.post(`/chat/${this.$route.params.id}`, {
        text: this.chat,
        username: this.currentUser,
      });
      this.chat = "";
      this.$refs.chatContainer.scrollTop = this.chatHeight + 5000;
    },
    getChatData() {
      this.$axios.get(`/chat/${this.$route.params.id}`).then((res) => {
        this.tmpChats = res.data;
      });
    },
  },
};
</script>

<style lang="sass" scoped>
.chat-container
  height: 81vh
  overflow-y: scroll

.chat-left
  display: inline-block
  position: relative
  width: 250px
  height: auto
  background-color: $accent--color

.chat-right
  display: inline-block
  position: relative
  width: 250px
  height: auto
  background-color: $primary--color
  color: white

.rounded-chat
  border-radius: 10px
  -webkit-border-radius: 10px
  -moz-border-radius: 10px

.chat-left.triangle-chat:before
  content: ' '
  position: absolute
  width: 0
  height: 0
  left: 0
  right: auto
  top: auto
  bottom: -20px
  border: 25px solid
  border-color: $accent--color transparent transparent $accent--color

.chat-right.triangle-chat:before
  content: ' '
  position: absolute
  width: 0
  height: 0
  left: auto
  right: 0
  top: auto
  bottom: -20px
  border: 25px solid
  border-color: transparent $primary--color transparent transparent

.chat-text
  padding: 1em
  text-align: left
  line-height: 1.5em
</style>
