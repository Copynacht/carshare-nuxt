<template>
  <v-container>
    <v-row>
      <v-col v-for="(car, key) in cars" :key="key">
        <v-card v-if="car.available === 0" @click="jumpReservation(car.id)">
          <v-card-title v-text="car.name" />
          <v-row no-gutters>
            <v-card-text class="py-0" v-text="'説明：' + car.description" />
            <v-card-text
              class="py-0"
              v-text="'走行距離：' + car.odometer + 'km'"
            />
            <v-card-text
              class="py-0"
              v-text="'使用回数：' + car.per_use + '回'"
            />
            <v-card-text
              class="py-0"
              v-text="'ミッション：' + mission[car.mission]"
            />
            <v-card-text
              class="pt-0"
              v-text="'保険：' + insurance[car.insurance]"
            />
          </v-row>
        </v-card>
        <v-card v-else color="grey darken-1">
          <v-card-title v-text="car.name" />
          <v-row no-gutters>
            <v-card-text class="py-0" v-text="'説明：' + car.description" />
            <v-card-text
              class="py-0"
              v-text="'走行距離：' + car.odometer + 'km'"
            />
            <v-card-text
              class="py-0"
              v-text="'使用回数：' + car.per_use + '回'"
            />
            <v-card-text
              class="py-0"
              v-text="'ミッション：' + mission[car.mission]"
            />
            <v-card-text
              class="pt-0"
              v-text="'保険：' + insurance[car.insurance]"
            />
          </v-row>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import axios from 'axios'

export default {
  name: 'Cars',
  data() {
    return {
      cars: [],
      width: 0,
      mission: ['AT', 'MT'],
      insurance: ['任意保険に加入済み', '１日保険に加入が必要'],
    }
  },
  mounted() {
    this.getCars()
  },
  methods: {
    jumpReservation(id) {
      this.$router.push('/reservation/' + id)
    },
    getCars() {
      axios
        .get(process.env.baseUrl + '/api/car/', {
          headers: {
            Authorization:
              'JWT ' + this.$auth.getToken('local').replace('Bearer', ''),
          },
        })
        .then((res) => {
          this.cars = res.data
        })
        .catch((err) => {
          return err
        })
    },
  },
}
</script>
