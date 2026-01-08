<template>
  <v-container>
    <v-row justify="center">
      <v-col cols="12" md="8">
        <v-card>
          <v-card-title class="text-center text-h5">채팅</v-card-title>
          <v-card-text>
            <div class="chat-box">
              <div
                v-for="(msg, index) in messages"
                :key="index"
                :class="[
                  'chat-message',
                  msg.senderEmail === senderEmail ? 'sent' : 'received',
                ]"
              >
                <strong>{{ msg.senderEmail }}: </strong>{{ msg.message }}
              </div>
            </div>
            <v-text-field
              v-model="newMessage"
              label="메시지 입력"
              @keyup.enter="sendMessage"
            />
            <v-btn color="primary" block @click="sendMessage">전송</v-btn>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import SockJS from "sockjs-client";
import Stomp from "webstomp-client";
//import axios from "axios";

export default {
  data() {
    return {
      messages: [], // 채팅 메시지 목록
      newMessage: "", // 입력중인 메시지
      stompClient: null, // 스톰프 연결 객체
      token: "",
      senderEmail: "", //로그인 사용자 이메일
      roomId: null,
    };
  },
  created() {
    this.senderEmail = localStorage.getItem("email"); // 로그인 사용자 이메일
    this.roomId = this.$route.params.roomId;
    this.connectWebsocket(); // 채팅 화면 들어오는 순간 웹소켓 즉시연결
  },
  // 사용자가 현재 라우트에서 다른 라우트로 이동하려고 할때 호출되는 훅함수
  beforeRouteLeave(to, from, next) {
    this.disconnectWebSocket();
    next();
  },
  // 화면을 완전히 떠날떄(꺼버렸을떄)
  beforeUnmount() {
    this.disconnectWebSocket();
  },
  methods: {
    connectWebsocket() {
      if (this.stompClient && this.stompClient.connected) return;

      // sockjs는 websocket을 내장한 향상된 js 라이브러리. http 엔드포인트 사용.
      const sockJs = new SockJS(
        `${import.meta.env.VITE_APP_API_BASE_URL}/connect`
      );

      this.stompClient = Stomp.over(sockJs);

      this.token = localStorage.getItem("token");

      this.stompClient.connect(
        {
          Authorization: `Bearer ${this.token}`, //STOMP CONNECT 헤더에 JWT 전달 , 서버 stompHandler 에서 검증 가능
        },
        () => {
          // 클라이언트 구독 , SimpleBroker가 구독자 목록에 등록
          this.stompClient.subscribe(`/topic/${this.roomId}`, (message) => {
            console.log(message); // ex "{"message":"ㅎㅇ","senderEmail":"limcheyean@gmail.com"}"
            const parseMessage = JSON.parse(message.body);
            this.messages.push(parseMessage);
            this.scrollToBottom();
          });
        }
      );
    },
    sendMessage() {
      if (this.newMessage.trim() === "") return;

      const messageData = {
        message: this.newMessage,
        senderEmail: this.senderEmail,
      };

      this.stompClient.send(
        `/publish/${this.roomId}`,
        JSON.stringify(messageData)
      );

      this.clearInput();
    },
    clearInput() {
      this.newMessage = "";
    },
    scrollToBottom() {
      this.$nextTick(() => {
        const chatBox = this.$el.querySelector(".chat-box");
        chatBox.scrollTop = chatBox.scrollHeight;
      });
    },
    disconnectWebSocket() {
      if (this.stompClient && this.stompClient.connected) {
        this.stompClient.unsubscribe(`/topic/${this.roomId}`);
        this.stompClient.disconnect();
      }
    },
  },
};
</script>

<style>
.chat-box {
  height: 300px;
  overflow-y: auto;
  border: 1px solid #ddd;
  margin-bottom: 10px;
}
.chat-message {
  margin-bottom: 10px;
}
.sent {
  text-align: right;
}
.received {
  text-align: left;
}
</style>
