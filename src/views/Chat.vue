<template>
  <div>
    <HeaderLogin style="padding: 20px 0" />
    <b-jumbotron class="mb-0">
      <b-container>
        <b-row>
          <b-col cols lg="4" sm="12" class="chat-room">
            <b-card style="height: 400px">
              <p>Chat</p>
              <hr />
              <div class="my-5" v-if="(room.room_name = 0)">
                <b-img
                  center
                  fluid
                  :src="require('@/assets/message.png')"
                  alt="Image 1"
                  class
                ></b-img>
                <p class="text-center" style="font-size: 15px">No New Inbox</p>
              </div>
              <div style="overflowy: scroll; height: 85%">
                <div
                  @click="getDataRoom(value)"
                  v-for="(value, index) in room"
                  :key="index"
                >
                  <div class="room mb-2">
                    <div class="img" style="cursor: pointer">
                      <img
                        v-if="value.image === null"
                        class="img"
                        src="../assets/img/default.png"
                        alt=""
                      />
                      <img
                        v-if="value.image !== null"
                        class="img"
                        :src="port + value.image"
                        alt=""
                      />
                    </div>
                    <h5 style="cursor: pointer">{{ value.room_name }}</h5>
                  </div>
                </div>
              </div>
            </b-card>
          </b-col>
          <b-col cols lg="8" sm="12">
            <b-card style="height: 600px; position: relative">
              <div>
                <div class="room mb-2">
                  <div
                    class="img"
                    v-if="
                      receiverImg === null ||
                      receiverImg === undefined ||
                      receiverImg === ''
                    "
                  >
                    <img class="img" src="../assets/img/default.png" alt="" />
                  </div>
                  <div class="img" v-if="receiverImg !== null">
                    <img class="img" :src="port + receiverImg" alt="" />
                  </div>
                  <h5 v-if="receiverName">{{ receiverName }}</h5>
                </div>
              </div>
              <hr />
              <div style="overflow-x: hidden; height: 73%">
                <div v-for="(value, index) in msg" :key="index">
                  <div class="chat-r">
                    <div class="img" v-if="value.sender_img === null">
                      <img class="img" src="../assets/img/default.png" alt="" />
                    </div>
                    <div class="img" v-if="value.sender_img !== null">
                      <img class="img" :src="port + value.sender_img" alt="" />
                    </div>
                    <h5>{{ value.sender_name }}</h5>
                  </div>
                  <p class="msg">{{ value.message_text }}</p>
                  <hr />
                </div>
              </div>

              <div class="send-chat">
                <b-form-textarea
                  type="text"
                  rows="1"
                  max-rows="1"
                  id="input-2"
                  required
                  v-model="message_text"
                  placeholder="Send a message"
                ></b-form-textarea>
                <b-button class="btn btn-send" @click="sendMessage"
                  >Send</b-button
                >
              </div>
            </b-card>
          </b-col>
        </b-row>
      </b-container>
    </b-jumbotron>
    <Footer />
  </div>
</template>

<script>
import HeaderLogin from '@/components/HeaderLogin'
import Footer from '@/components/Footer'
import { mapGetters, mapActions } from 'vuex'

export default {
  name: 'Chat',
  data() {
    return {
      userId: '',
      senderName: '',
      message_text: '',
      port: `${process.env.VUE_APP_URL}profile/`,
      receiverName: '',
      receiverImg: ''
    }
  },
  components: {
    HeaderLogin,
    Footer
  },
  computed: {
    ...mapGetters({ dataLogin: 'getUser', room: 'dataRoom', msg: 'messages' })
  },
  created() {
    this.getIdUser()
  },
  methods: {
    ...mapActions([
      'getWorkerRoom',
      'getRoomById',
      'getRecruiterRoom',
      'postMessage'
    ]),
    getIdUser() {
      if (this.dataLogin.role === 1) {
        this.userId = this.dataLogin.recruiter_id
        this.getRecruiterRoom(this.userId)
      } else if (this.dataLogin.role === 2) {
        this.userId = this.dataLogin.user_id
        this.getWorkerRoom(this.userId)
      }
    },
    getDataRoom(data) {
      this.getRoomById(data.room_id)
      this.receiverImg = data.image
      this.receiverName = data.room_name
    },
    sendMessage() {
      if (this.dataLogin.role === 2) {
        const sendMail = {
          recruiter_id: this.room[0].recruiter_id,
          user_id: this.dataLogin.user_id,
          role: this.dataLogin.role,
          sender_id: this.dataLogin.user_id,
          message_text: this.message_text
        }
        this.postMessage(sendMail)
          .then((result) => {
            console.log(result)
          })
          .catch((error) => error)
      } else {
        const sendMail = {
          user_id: this.room[0].user_id,
          recruiter_id: this.dataLogin.recruiter_id,
          role: this.dataLogin.role,
          sender_id: this.dataLogin.recruiter_id,
          message_text: this.message_text
        }
        this.postMessage(sendMail)
          .then((result) => {
            console.log(result)
          })
          .catch((error) => error)
      }
    }
  }
}
</script>

<style scoped>
.send-chat {
  display: flex;
  position: absolute;
  width: 100%;
  gap: 10px;
  margin-left: 5px;
  bottom: 5px;
  left: 0px;
}

.btn-send {
  background-color: #fbb017;
  border-color: #fbb017;
  width: 20%;
  margin-right: 20px;
}

.chat-room {
  height: 1000px;
}

.chat-r {
  display: grid;
  grid-template-columns: 40px 1fr;
  align-items: center;
}

.msg {
  margin: 10px 5px;
  padding-left: 10px;
  padding-top: 3px;
  padding-bottom: 5px;
  color: white;
  border-radius: 4px;
  background-color: #5e50a1;
}

.room {
  width: 100%;
  display: grid;
  grid-template-columns: 40px 1fr;
  align-items: center;
}

.img {
  /* margin-left: 20px; */
  width: 85%;
  height: 30px;
  /* background-color: #111; */
  border-radius: 50%;
  /* display: inline-block; */
}

h5 {
  text-align: left;
  margin-left: 4px;
  font-size: 17px;
  margin-top: 5px;
}

@media (max-width: 600px) {
  .chat-room {
    height: 450px;
  }
}
</style>
