<template>
  <div id="signin">
    <v-container>
      <v-row justify="center" align-content="center">
        <v-col cols="12" sm="8" md="6" lg="5" xl="3">
          <v-card>
            <v-card-title>
              <span class="headline">ログイン</span>
            </v-card-title>

            <v-spacer />

            <v-card-text>
              <v-form ref="form" v-model="valid" lazy-validation>
                <v-container>
                  <v-text-field
                    v-model="credentials.username"
                    :counter="100"
                    label="ユーザー名"
                    :rules="rules.username"
                    maxlength="100"
                    required
                    autofocus
                  />

                  <v-text-field
                    v-model="credentials.password"
                    type="password"
                    :counter="100"
                    label="パスワード"
                    :rules="rules.password"
                    maxlength="100"
                    required
                  />
                </v-container>
              </v-form>
              <v-card-actions class="mx-7 my-2">
                <v-btn
                  color="primary"
                  :disabled="!valid"
                  depressed
                  block
                  large
                  @click="login"
                >
                  <h4>ログイン</h4>
                </v-btn>
              </v-card-actions>
              <div align="center">
                <h5>
                  <div>
                    アカウントをお持ちでない方は
                    <nuxt-link :to="`/signup`">こちら</nuxt-link>から
                  </div>
                </h5>
              </div>
            </v-card-text>
          </v-card>
        </v-col>
      </v-row>
    </v-container>
  </div>
</template>

<script>
import Swal from 'sweetalert2'
export default {
  name: 'Signin',
  layout: 'sign',
  auth: false,
  middleware({ store, redirect }) {
    if (store.$auth.loggedIn) {
      redirect('/')
    }
  },
  data: () => ({
    credentials: {},
    valid: false,
    rules: {
      username: [(value) => !!value || 'ユーザー名を入力してください'],
      password: [(value) => !!value || 'パスワードを入力してください'],
    },
  }),
  methods: {
    login() {
      if (this.$refs.form.validate()) {
        this.$auth
          .loginWith('local', {
            data: {
              username: this.credentials.username,
              password: this.credentials.password,
            },
          })
          .then((res) => {
            this.$router.push('/')
          })
          .catch((e) => {
            Swal.fire({
              title: 'Error',
              text: e.response.data.non_field_errors,
              showConfirmButton: false,
              showCloseButton: false,
              timer: 3000,
            })
          })
      }
    },
  },
}
</script>
