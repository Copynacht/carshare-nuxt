<template>
  <v-container>
    <v-row justify="center">
      <v-col cols="12" sm="8" md="6" lg="5" xl="3">
        <v-card>
          <client-only>
            <v-card-title>
              <span class="headline">予約</span>
            </v-card-title>
            <v-spacer />
            <v-container class="px-5 pb-5">
              <v-row justify="center">
                <v-col class="pb-2" cols="12">
                  <VueCtkDateTimePicker
                    v-model="startDate"
                    label="利用開始日時"
                    :minute-interval="15"
                    :format="'YYYY-MM-DD HH:mm'"
                    :overlay="true"
                    :min-date="start"
                    :max-date="end"
                  ></VueCtkDateTimePicker>
                </v-col>
                <v-col class="pb-2" cols="12">
                  <VueCtkDateTimePicker
                    v-model="endDate"
                    label="返却日時"
                    :minute-interval="15"
                    :format="'YYYY-MM-DD HH:mm'"
                    :overlay="true"
                    :min-date="start"
                    :max-date="end"
                  ></VueCtkDateTimePicker>
                </v-col>
                <v-btn class="blue--text" @click="reserve()"> 更新 </v-btn>
              </v-row>
            </v-container>
          </client-only>
        </v-card>
      </v-col>

      <v-col cols="12">
        <v-card>
          <Calendar :reservations="reservations" />
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import axios from 'axios'
import Swal from 'sweetalert2'
import moment from 'moment'
import Calendar from '~/components/Calendar.vue'

export default {
  name: 'Thread',
  components: {
    Calendar,
  },
  data() {
    return {
      startDate: '',
      endDate: '',
      reservations: [],
      reservation: {},
      reservationID: 0,
      carID: 0,
    }
  },
  computed: {
    start() {
      const start = moment()
      return start.format('YYYY-MM-DDTHH:mm:ss')
    },
    end() {
      const start = moment(this.start)
      const end = start.add(3, 'months').endOf('day')
      return end.format('YYYY-MM-DDTHH:mm:ss')
    },
  },
  mounted() {
    this.reservationID = this.$router.currentRoute.params.id
    this.getReservation()
  },
  methods: {
    getReservation() {
      axios
        .get(
          process.env.baseUrl + '/api/reservation/' + this.reservationID + '/',
          {
            headers: {
              Authorization:
                'Bearer ' + this.$auth.getToken('local').replace('Bearer', ''),
            },
          }
        )
        .then((res) => {
          this.reservation = res.data
          this.carID = this.reservation.car.id
          this.startDate = moment(this.reservation.start_date_time).format(
            'yyyy-MM-DD HH:mm'
          )
          this.endDate = moment(this.reservation.end_date_time).format(
            'yyyy-MM-DD HH:mm'
          )
          axios
            .get(
              process.env.baseUrl +
                '/api/reservation/?status=0&car=' +
                this.carID,
              {
                headers: {
                  Authorization:
                    'Bearer ' + this.$auth.getToken('local').replace('Bearer', ''),
                },
              }
            )
            .then((res) => {
              this.reservations = res.data
              const r = []
              this.reservations.forEach(function (elem) {
                r.push({
                  name: elem.user.username,
                  start: moment(elem.start_date_time).toDate(),
                  end: moment(elem.end_date_time).toDate(),
                  color: 'red',
                  timed: true,
                  email: elem.user.email,
                  se:
                    moment(elem.start_date_time).format('yyyy/MM/DD HH:mm') +
                    ' ～ ' +
                    moment(elem.end_date_time).format('yyyy/MM/DD HH:mm'),
                })
              })
              this.reservations = r
            })
            .catch((err) => {
              return err
            })
        })
        .catch((err) => {
          return err
        })
    },
    getReservations() {
      axios
        .get(
          process.env.baseUrl + '/api/reservation/?status=0&car=' + this.carID,
          {
            headers: {
              Authorization:
                'Bearer ' + this.$auth.getToken('local').replace('Bearer', ''),
            },
          }
        )
        .then((res) => {
          this.reservations = res.data
          const r = []
          this.reservations.forEach(function (elem) {
            r.push({
              name: elem.user.username,
              start: moment(elem.start_date_time).toDate(),
              end: moment(elem.end_date_time).toDate(),
              color: 'red',
              timed: true,
              email: elem.user.email,
              se:
                moment(elem.start_date_time).format('yyyy/MM/DD HH:mm') +
                ' ～ ' +
                moment(elem.end_date_time).format('yyyy/MM/DD HH:mm'),
            })
          })
          this.reservations = r
        })
        .catch((err) => {
          return err
        })
    },
    reserve() {
      if (this.startDate === '' || this.endDate === '') {
        Swal.fire({
          title: 'Error',
          text: '利用開始予定日時と返却予定日時を入力してください',
          showConfirmButton: false,
          showCloseButton: false,
          timer: 3000,
        })
        return
      }
      axios
        .patch(
          process.env.baseUrl +
            '/api/reservation/' +
            this.reservationID +
            '/change/',
          {
            start_date_time: moment(this.startDate).format(),
            end_date_time: moment(this.endDate).format(),
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
            title: 'お知らせ',
            text: '予約を変更しました',
            showConfirmButton: false,
            showCloseButton: false,
            timer: 3000,
          })
          this.$router.push('/')
          return res
        })
        .catch((err) => {
          Swal.fire({
            title: 'Error',
            text: err.response.data,
            showConfirmButton: false,
            showCloseButton: false,
            timer: 3000,
          })
        })
    },
  },
}
</script>
