<template>
  <v-app-bar app dark>
    <v-container>
      <v-row align="center">
        <v-col class="d-flex justify-start">
          <v-btn :to="{ path: '/member/list' }">회원 목록</v-btn>
          <v-btn :to="{ path: '/groupchatting/list' }">채팅방 목록</v-btn>
        </v-col>

        <v-col class="text-center">
          <v-btn :to="{ path: '/' }"> chat 서비스 </v-btn>
        </v-col>

        <v-col class="d-flex justify-end">
          <v-btn v-if="isLogin" :to="{ path: '/my/chat/page' }"
            >MyChatPage</v-btn
          >
          <v-btn v-if="!isLogin" :to="{ path: '/member/create' }"
            >회원가입</v-btn
          >
          <v-btn v-if="!isLogin" :to="{ path: '/login' }">로그인</v-btn>
          <v-btn v-if="isLogin" @click="doLogout">로그아웃</v-btn>
        </v-col>
      </v-row>
    </v-container>
  </v-app-bar>
</template>

<script>
export default {
  computed: {
    isLogin() {
      return !!localStorage.getItem("token");
    },
  },
  methods: {
    doLogout() {
      localStorage.removeItem("token");
      // this.$router.push("/");
      window.location.reload();
    },
  },
};
</script>

<style scoped></style>
