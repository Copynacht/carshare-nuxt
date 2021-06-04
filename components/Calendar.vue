<template>
  <div>
    <v-dialog v-model="dialog" width="500">
      <v-card>
        <v-card-title>{{ dtitle }} </v-card-title>
        <v-divider />
        <v-sheet class="px-5 py-5">
          <v-col class="py-1">
            {{ dcontent }}
          </v-col>
          <v-col class="py-1">
            {{ daddress }}
          </v-col>
        </v-sheet>
      </v-card>
    </v-dialog>

    <v-sheet tile height="10vh" color="white" class="d-flex align-center">
      <v-btn outlined small class="ma-4" @click="setToday"> 今日 </v-btn>
      <v-toolbar-title>{{ title }}</v-toolbar-title>

      <v-btn icon @click="$refs.calendar.prev()">
        <v-icon>mdi-chevron-left</v-icon>
      </v-btn>
      <v-btn icon @click="$refs.calendar.next()">
        <v-icon>mdi-chevron-right</v-icon>
      </v-btn>
    </v-sheet>
    <v-sheet height="90vh">
      <v-calendar
        ref="calendar"
        v-model="value"
        :timezone="'Asia/Tokyo'"
        :events="reservations"
        :event-color="getEventColor"
        locale="ja-jp"
        :day-format="(timestamp) => new Date(timestamp.date).getDate()"
        :month-format="
          (timestamp) => new Date(timestamp.date).getMonth() + 1 + ' /'
        "
        type="week"
        @click:event="showEvent"
      >
        <template #day-body="{ date, week }">
          <div
            class="v-current-time"
            :class="{ first: date === week[0].date }"
            :style="{ top: nowY }"
          ></div>
        </template>
      </v-calendar>
    </v-sheet>
  </div>
</template>

<script>
import moment from 'moment'

export default {
  props: {
    reservations: {
      type: Array,
      required: false,
      default: () => [],
    },
  },
  data: () => ({
    events: [],
    value: moment().format('yyyy-MM-DD'),
    dialog: false,
    dtitle: '',
    dcontent: '',
    daddress: '',
    ready: false,
  }),
  computed: {
    title() {
      return moment(this.value).format('yyyy年 M月')
    },
    cal() {
      return this.ready ? this.$refs.calendar : null
    },
    nowY() {
      return this.cal ? this.cal.timeToY(this.cal.times.now) + 'px' : '-10px'
    },
  },
  mounted() {
    this.ready = true
    this.scrollToTime()
    this.updateTime()
  },
  methods: {
    getCurrentTime() {
      return this.cal
        ? this.cal.times.now.hour * 60 + this.cal.times.now.minute
        : 0
    },
    scrollToTime() {
      const time = this.getCurrentTime()
      const first = Math.max(0, time - (time % 30) - 30)
      this.cal.scrollToTime(first)
    },
    updateTime() {
      setInterval(() => this.cal.updateTimes(), 60 * 1000)
    },
    setToday() {
      this.value = moment().format('yyyy-MM-DD')
    },
    showEvent({ event }) {
      this.dtitle = event.name + 'さんによる予約です'
      this.dcontent = '予約期間：' + event.se
      this.daddress = '連絡先：' + event.email
      this.dialog = true
    },
    getEventColor(event) {
      return event.color
    },
  },
}
</script>

<style lang="scss">
.v-current-time {
  height: 2px;
  background-color: limegreen;
  position: absolute;
  left: -1px;
  right: 0;
  pointer-events: none;
  &.first::before {
    content: '';
    position: absolute;
    background-color: limegreen;
    width: 12px;
    height: 12px;
    border-radius: 50%;
    margin-top: -5px;
    margin-left: -6.5px;
  }
}
</style>
