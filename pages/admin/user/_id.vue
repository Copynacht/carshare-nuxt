<template>
  <v-container>
    <v-row>
      <v-col v-for="(eco, key1) in eachCarOdometer" :key="key1" cols="4">
        <v-card>
          <v-card-title v-text="eco.car + '：' + eco.odo + 'km'" />
        </v-card>
      </v-col>
      <v-col v-for="(mr, key2) in myres" :key="key2 + 20" cols="12">
        <v-card>
          <v-card-title
            v-text="
              '走行距離：' +
              (Number(mr.end_odometer) - Number(mr.start_odometer))
            "
          />
          <v-row>
            <v-card-text class="py-0" v-text="'車種：' + mr.car.name" />
            <v-card-text class="py-0" v-text="'開始予定時刻：' + mr.start" />
            <v-card-text class="pt-0" v-text="'返却予定時刻：' + mr.end" />
          </v-row>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import axios from 'axios'
import moment from 'moment'

export default {
  name: 'UserReservation',
  data() {
    return {
      myres: [],
      cars: [],
      eachCarOdometer: [],
    }
  },
  mounted() {
    this.getEachCar()
    this.getReservations()
  },
  methods: {
    getEachCar() {
      axios
        .get(process.env.baseUrl + '/api/car/', {
          headers: {
            Authorization:
              'JWT ' + this.$auth.getToken('local').replace('Bearer', ''),
          },
        })
        .then((res) => {
          this.cars = res.data
          for (const c in this.cars) {
            axios
              .get(
                process.env.baseUrl +
                  '/api/reservation/?user=' +
                  this.$router.currentRoute.params.id +
                  '&status=2&pay=false&car=' +
                  this.cars[c].id,
                {
                  headers: {
                    Authorization:
                      'JWT ' +
                      this.$auth.getToken('local').replace('Bearer', ''),
                  },
                }
              )
              .then((res) => {
                const carres = res.data
                let odometer = 0
                for (const cr in carres) {
                  odometer +=
                    carres[cr].end_odometer - carres[cr].start_odometer
                }
                this.eachCarOdometer.push({
                  car: this.cars[c].name,
                  odo: odometer,
                })
              })
              .catch((err) => {
                return err
              })
          }
        })
        .catch((err) => {
          return err
        })
    },

    getReservations() {
      axios
        .get(
          process.env.baseUrl +
            '/api/reservation/?user=' +
            this.$router.currentRoute.params.id +
            '&status=2&pay=false',
          {
            headers: {
              Authorization:
                'JWT ' + this.$auth.getToken('local').replace('Bearer', ''),
            },
          }
        )
        .then((res) => {
          this.myres = res.data
          const mr = []
          this.myres.forEach(function (elem) {
            mr.push({
              id: elem.id,
              car: elem.car,
              start: moment(elem.start_date_time).format('yyyy/MM/DD HH:mm'),
              end: moment(elem.end_date_time).format('yyyy/MM/DD HH:mm'),
              status: elem.status,
              start_odometer: elem.start_odometer,
              end_odometer: elem.end_odometer,
            })
          })
          this.myres = mr
        })
        .catch((err) => {
          return err
        })
    },
  },
}
</script>
