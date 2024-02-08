<template>
  <div id="mypage">
    <v-container>
      <v-row class="button" justify="center" align-content="center">
        <v-col cols="12" sm="8" md="6" lg="5" xl="3">
          <v-card>
            <v-form ref="form" v-model="valid" lazy-validation>
              <v-card-title>
                <span class="headline">登録情報</span>
              </v-card-title>
              <v-spacer />
              <v-card-text>
                <v-container>
                  <v-text-field
                    v-model="currentUser.username"
                    :counter="10"
                    label="ユーザー名(変更可)"
                    maxlength="100"
                    required
                    :rules="rules.username"
                  />
                  <v-text-field
                    v-model="currentUser.email"
                    label="メールアドレス(変更可)"
                    maxlength="100"
                    readonly
                    required
                  />
                </v-container>
                <v-btn
                  class="blue--text"
                  :disabled="!valid"
                  @click="updateMyPage()"
                >
                  更新
                </v-btn>
              </v-card-text>
            </v-form>
          </v-card>
        </v-col>
      </v-row>
    </v-container>
  </div>
</template>

<script>
import Swal from 'sweetalert2'
import axios from 'axios'
export default {
  name: 'MyPage',
  data: () => ({
    dialog: false,
    currentUser: {},
    valid: false,
    rules: {
      username: [
        (value) => !!value || 'ユーザー名を入力してください',
        (value) => !!value || 'メールアドレスを入力してください',
      ],
    },
  }),
  mounted() {
    this.getMyPage()
  },
  methods: {
    getMyPage() {
      axios
        .get(process.env.baseUrl + '/user/mypage/', {
          headers: {
            Authorization:
              'Bearer ' + this.$auth.getToken('local').replace('Bearer', ''),
          },
        })
        .then((res) => {
          this.currentUser = res.data
        })
        .catch((err) => {
          return err
        })
    },
    updateMyPage() {
      axios
        .patch(
          process.env.baseUrl + '/user/mypage/',
          {
            username: this.currentUser.username,
            email: this.currentUser.email,
          },
          {
            headers: {
              Authorization:
                'Bearer ' + this.$auth.getToken('local').replace('Bearer', ''),
            },
          }
        )
        .then((res) => {
          Swal.fire({
            title: 'Success',
            text: 'プロフィールを更新しました。再ログインしてください。',
            showConfirmButton: false,
            showCloseButton: false,
            timer: 3000,
          })
          this.$router.push('/signout')
          return res
        })
        .catch((err) => {
          Swal.fire({
            title: 'Error',
            text: err.response.data.username,
            showConfirmButton: false,
            showCloseButton: false,
            timer: 3000,
          })
        })
    },
  },
}
</script>

<style>
.button {
  text-align: center;
}
</style>
