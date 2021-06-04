<template>
  <v-container>
    <v-dialog v-model="cancelDialog" persistent max-width="400px">
      <v-card>
        <v-card-title>
          <span class="headline">警告</span>
        </v-card-title>
        <v-card-text>本当にキャンセルしますか？</v-card-text>
        <v-card-actions>
          <v-spacer />
          <v-btn
            color="red"
            class="dialog-Button"
            @click="cancelReservation(cancelID)"
          >
            キャンセルする
          </v-btn>
          <v-btn class="dialog-Button" @click="closeDialog()"> 取りやめ </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
    <v-dialog v-model="returnDialog" persistent max-width="400px">
      <v-card>
        <v-sheet class="px-5 py-5">
          <v-card-title>
            <span class="headline">返却</span>
          </v-card-title>
          <v-row class="px-5">
            <v-col cols="12">
              <v-text-field
                v-model="startOdometer"
                label="利用前の距離メーター"
                required
                autofocus
              />
            </v-col>
            <v-col cols="12">
              <v-text-field
                v-model="endOdometer"
                label="利用後の距離メーター"
                required
              />
            </v-col>
          </v-row>
          <v-card-actions>
            <v-spacer />
            <v-btn class="dialog-Button" @click="returnCar()"> 返却 </v-btn>
            <v-btn class="dialog-Button" @click="closeReturnDialog()">
              取りやめ
            </v-btn>
          </v-card-actions>
        </v-sheet>
      </v-card>
    </v-dialog>

    <v-row>
      <v-col v-for="(mr, key) in myres" :key="key" cols="12">
        <v-card v-if="mr.status === 0">
          <v-card-title v-text="status[mr.status]" />
          <v-row>
            <v-col cols="10">
              <v-row no-gutters>
                <v-card-text class="py-0" v-text="'車種：' + mr.car.name" />
                <v-card-text
                  class="py-0"
                  v-text="'開始予定時刻：' + mr.start"
                />
                <v-card-text class="pt-0" v-text="'返却予定時刻：' + mr.end" />
              </v-row>
            </v-col>
            <v-col cols="2" justify="end">
              <div class="my-2">
                <v-btn
                  color="primary"
                  fab
                  small
                  dark
                  @click="openReturnDialog(mr.id)"
                >
                  <v-icon>mdi-keyboard-return</v-icon>
                </v-btn>
              </div>
              <div class="my-2">
                <v-btn
                  color="primary"
                  fab
                  small
                  dark
                  @click="changeReservation(mr.id)"
                >
                  <v-icon>mdi-pencil</v-icon>
                </v-btn>
              </div>
              <div class="my-2">
                <v-btn
                  color="primary"
                  fab
                  small
                  dark
                  @click="openDialog(mr.id)"
                >
                  <v-icon>mdi-close</v-icon>
                </v-btn>
              </div>
            </v-col>
          </v-row>
        </v-card>
        <v-card v-else>
          <v-card-title v-text="status[mr.status]" />
          <v-row no-gutters>
            <v-card-text class="py-0" v-text="'車種：' + mr.car.name" />
            <v-card-text
              class="py-0"
              v-text="
                '利用距離：' + (mr.end_odometer - mr.start_odometer) + 'km'
              "
            />
            <v-card-text class="py-0" v-text="'開始時刻：' + mr.start" />
            <v-card-text class="pt-0" v-text="'返却時刻：' + mr.end" />
          </v-row>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import axios from 'axios'
import Swal from 'sweetalert2'
import moment from 'moment'

export default {
  name: 'Cars',
  data() {
    return {
      cancelID: 0,
      cancelDialog: false,
      returnID: 0,
      returnDialog: false,
      startOdometer: 0,
      endOdometer: 0,
      myres: [],
      status: ['予約中', '予約キャンセル', '返却済み', '事故'],
    }
  },
  mounted() {
    this.getReservations()
  },
  methods: {
    returnCar() {
      if (this.startOdometer === 0 || this.endOdometer === 0) {
        Swal.fire({
          title: 'Error',
          text: '距離メーターの値を入力してください',
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
            this.returnID +
            '/returncar/',
          {
            start_odometer: this.startOdometer,
            end_odometer: this.endOdometer,
          },
          {
            headers: {
              Authorization:
                'JWT ' + this.$auth.getToken('local').replace('Bearer', ''),
            },
          }
        )
        .then((res) => {
          Swal.fire({
            title: 'お知らせ',
            text: '返却を完了しました',
            showConfirmButton: false,
            showCloseButton: false,
            timer: 3000,
          })
          this.returnDialog = false
          this.getReservations()
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
    openDialog(id) {
      this.cancelID = id
      this.cancelDialog = true
    },
    closeDialog() {
      this.cancelDialog = false
    },
    changeReservation(id) {
      this.$router.push('/change/' + id)
    },
    cancelReservation(id) {
      this.cancelDialog = false
      axios
        .patch(
          process.env.baseUrl + '/api/reservation/' + id + '/cancel/',
          {},
          {
            headers: {
              Authorization:
                'JWT ' + this.$auth.getToken('local').replace('Bearer', ''),
            },
          }
        )
        .then((res) => {
          this.getReservations()
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
    jumpReservation(id) {
      this.$router.push('/reservation/' + id)
    },
    openReturnDialog(id) {
      this.returnID = id
      this.returnDialog = true
    },
    closeReturnDialog() {
      this.returnDialog = false
    },
    getReservations() {
      axios
        .get(process.env.baseUrl + '/api/reservation/?my=true', {
          headers: {
            Authorization:
              'JWT ' + this.$auth.getToken('local').replace('Bearer', ''),
          },
        })
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
