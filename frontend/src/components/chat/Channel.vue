<template>
  <div class="channel" style="height: 100%"> 
    <channel-users style="height:5rem" v-if="choiceChannel" />
    <!-- <hr /> -->
    <messages style="height:70vh" v-if="choiceChannel" />

    <message-sender v-if="choiceChannel" />
  </div>
</template>

<script>
import sendBird from "@/services/SendBird.js";
import Messages from "@/components/chat/Messages";
import MessageSender from "@/components/chat/MessageSender";
import ChannelUsers from "@/components/chat/ChannelUsers";
import { mapState } from "vuex";

export default {
  name: "Channel",

  data(){
    return{
      choiceChannel:{},
    };
  },

  components: {
    Messages,
    MessageSender,
    ChannelUsers,
  },

  computed: {
    ...mapState(["channel"]),
  },

  watch: {
    channel: {
      async handler(newValue) {
        this.choiceChannel = newValue;
      },
    },
  },

  methods: {
    async init(url) {
      await sendBird
        .getChannel(url);
    },
  },
};
</script>

<style scoped lang="scss">
@import "../../assets/scss/index.scss";
</style>
