<template>
  <b-container class="bv-example-row mt-3 text-center">
    <b-row class="pt-3">
      <!-- 채팅 리스트 -->
      <b-col cols="3">
        <b-row>
          <b-card class="w-100">
            <b-row><br /></b-row>
            <b-row>
              <b-col> <b-icon icon="people-fill" /> 채팅 연결</b-col>
            </b-row>
            <b-row><br /></b-row>
          </b-card>
        </b-row>
        <b-row>
          <b-list-group class="w-100">
            <!-- 불러온 채팅 목록을 반복문을 통해 화면에 출력 -->
            <template v-for="room in chatroomlist">
              <!-- 클릭했을 때 해당 채팅방의 번호와 상대방의 아이디로 채팅 내역을 불러오고 채팅 리스트 router를 실행-->
              <b-list-group-item
                :key="room.no"
                button
                @click="
                  loadChatHistory(
                    room.no,
                    room.user_id == loginUser.id ? room.seller_id : room.user_id
                  ),
                    //새 채팅 알람 아이콘을 무조건 비활성화함
                    changeIconState(room),
                    //키 값을 변경해서 강제로 라우터를 새로고침
                    forceRerender()
                "
              >
                <!-- 상대방 아이디를 표시 -->
                <template v-if="room.seller_id != loginUser.id">
                  {{ room.seller_id }}
                </template>
                <template v-else>
                  {{ room.user_id }}
                </template>
                <!-- 새로운 채팅 내역이 도착하면 아이콘으로 표시 -->
                <template
                  v-if="
                    room.user_id == loginUser.id && room.userside_alert == 1
                  "
                >
                  <b-icon icon="messenger" variant="success"></b-icon>
                </template>
                <template
                  v-else-if="
                    room.seller_id == loginUser.id && room.sellerside_alert == 1
                  "
                >
                  <b-icon icon="messenger" variant="success"></b-icon>
                </template>
              </b-list-group-item>
              <!-- </router-link> -->
            </template>
          </b-list-group>
        </b-row>
      </b-col>
      <!-- 오른쪽 8 size의 채팅창 자리 생성-->
      <b-col cols="8">
        <b-jumbotron style="height: 800px">
          <div>
            <b-row>
              <b-col>
                <!-- 키를 설정해두고 키가 변경되면 강제로 router가 새로고침됨 -->
                <router-view :key="ComputedComponentKey" />
              </b-col>
            </b-row>
          </div>
        </b-jumbotron>
      </b-col>
    </b-row>
  </b-container>
</template>
<script>
import { mapGetters, mapActions } from "vuex";
import http from "@/api/http.js";
export default {
  name: "BoardView",
  data() {
    return { chatroomlist: "", componentKey: 0 };
  },
  computed: {
    ...mapGetters(["loginUser"]),
    ComputedComponentKey() {
      return this.componentKey;
    },
  },
  methods: {
    ...mapActions(["getChatHistory"]),
    forceRerender() {
      this.componentKey++;
    },
    changeIconState(room) {
      if (room.seller_id == this.loginUser.id) room.sellerside_alert = 0;
      else room.userside_alert = 0;
    },

    //웹 서버에 사용자가 포함된 채팅 목록을 불러옴
    async LoadChatRoomList() {
      await http
        .get(`/chat/${this.loginUser.id}/${this.loginUser.grade}`)
        .then((res) => {
          console.log("LoadChatRoomList : ", res.data);
          //불러온 데이터를 data()에 저장
          this.chatroomlist = res.data;
        })
        .catch((err) => {
          console.log(err);
        });
    },
    async loadChatHistory(no, user_id) {
      await http
        .get(`/chat/${no}`)
        .then((res) => {
          this.getChatHistory(res.data);
          // this.chatHistory = res.data;
          this.$router.push({
            name: "chatList",
            params: { no: no, receiver: user_id },
          });
        })
        .catch((err) => {
          console.log("loadChatHistory err : ", err);
        });
    },
  },

  created() {
    console.log("chatView created");
    if (!this.loginUser.id) {
      alert("로그인이 필요합니다.");
      this.$router.push({ name: "signIn" });
    } else {
      this.LoadChatRoomList();
    }
  },
};
</script>
<style scoped>
.underline-hotpink {
  display: inline-block;
  background: linear-gradient(
    180deg,
    rgba(255, 255, 255, 0) 70%,
    rgba(231, 27, 139, 0.3) 30%
  );
}
</style>
