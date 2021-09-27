<template>
  <v-container>
    <v-row>
      <v-col v-for="(user, key) in users" :key="key" cols="6">
        <v-card @click="jump(user.id)">
          <v-card-title v-text="user.username" />
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import axios from 'axios'

export default {
  name: 'AdminUser',
  data() {
    return {
      users: [],
    }
  },
  mounted() {
    this.getUsers()
  },
  methods: {
    jump(id) {
      this.$router.push('/admin/user/' + id)
    },
    getUsers() {
      axios
        .get(process.env.baseUrl + '/api/user/', {
          headers: {
            Authorization:
              'JWT ' + this.$auth.getToken('local').replace('Bearer', ''),
          },
        })
        .then((res) => {
          this.users = res.data
        })
        .catch((err) => {
          return err
        })
    },
  },
}
</script>
