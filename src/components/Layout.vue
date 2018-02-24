<template>
  <div>
    <q-layout ref="layout" view="lHh lpR fFr" :right-breakpoint="1100">
      <!-- Header -->
      <q-toolbar slot="header">
        <q-toolbar-title>
          Quasar Todo
          <span slot="subtitle">NewUp Developments</span>
        </q-toolbar-title>
      </q-toolbar>


      <router-view />

      <!-- Fixed Floating Point Button -->
      <q-fixed-position corner="bottom-right" :offset="[18, 18]" style="z-index: 1">
        <q-fab color="secondary" glossy icon="keyboard_arrow_left" direction="left" ref="fab">
          <q-fab-action color="secondary" icon="alarm" @click="showTime" title="Clock" />
          <q-fab-action color="secondary" icon="note" @click="openMemo" title="Memo" />
        </q-fab>
      </q-fixed-position>

      <!-- Memo Modal -->
      <q-modal ref="memoModal" minimized :content-css="{padding: '25px'}">
        <!-- Textarea -->
        <q-input v-model="memo" type="textarea" float-label="Memo" :max-height="100" :min-rows="7" />
        <q-btn color="yellow-8" @click="saveMemo">Save</q-btn>
        <q-btn color="secondary" @click="$refs.memoModal.close()">Close</q-btn>
      </q-modal>

      <!-- Time Modal -->
      <q-modal ref="timeModal" position="bottom" :content-css="clockStyle" class="clock">
        <p class="date">{{ date }}</p>
        <p class="time">{{ time }}</p>
      </q-modal>

      <!-- Footer -->
      <q-toolbar slot="footer" style="padding: 0px;">
        <q-search inverted v-model="query" style="box-shadow: none;" @input="searchItems" />
      </q-toolbar>
    </q-layout>

  </div>
</template>

<script>
  /*eslint-disable*/
  import { EventBus } from '../main.js'

  import { QLayout, QToolbar, QToolbarTitle, QBtn, QIcon, QTabs, QRouteTab, QList, QListHeader, QSideLink, QItemSide, QItemMain, QSearch, QFixedPosition, QFab, QFabAction, QModal, QInput } from 'quasar'

  export default {
    components: {
      QLayout, QToolbar, QToolbarTitle, QBtn, QIcon, QTabs, QRouteTab, QList, QListHeader, QSideLink, QItemSide, QItemMain, QSearch, QFixedPosition, QFab, QFabAction, QModal, QInput
    },
    data: () => ({
      query: '',
      memo: '',
      date: '',
      time: '',
      week: ['SUN', 'MON', 'TUE', 'WED', 'THU', 'FRI', 'SAT'],
      clockStyle: {
        paddingTop: '20px',
        color: '#daf6ff',
        backgroundColor: '#0f3854',
        background: 'radial-gradient(ellipse at center, #0a2e38  0%, #000000 80%)',
        borderRadius: '20px 20px 0 0'
      }
    }),
    computed: {

    },
    methods: {
      openMemo () {
        this.memo = localStorage.getItem('q-saved-memo')
        this.$refs.memoModal.open()
      },
      saveMemo () {
        localStorage.setItem('q-saved-memo', this.memo)
        this.$refs.memoModal.close()
      },
      searchItems () {
        EventBus.$emit('searching', this.query)
      },
      showTime () {
        this.$nextTick(() => {
          this.$refs.timeModal.open()
        })
      },
      zeroPadding (num, digit) {
        let zero = '';
        for (var i = 0; i < digit; i++) {
          zero += '0';
        }
        return (zero + num).slice(-digit)
      },
      updateTime () {
        let currentDate = new Date()

        this.time = this.zeroPadding(currentDate.getHours(), 2) + ':' + this.zeroPadding(currentDate.getMinutes(), 2) + ':' + this.zeroPadding(currentDate.getSeconds(), 2)

        this.date = this.week[currentDate.getDay()] + ' ' + this.zeroPadding(currentDate.getMonth()+ 1, 2) + '-' + this.zeroPadding(currentDate.getDate(), 2) + '-' + this.zeroPadding(currentDate.getFullYear(), 4)
      }
    },
    mounted () {
      let updateTime = this.updateTime
      let timeInt = setInterval(updateTime, 1000)
    }
  }
</script>

<style lang="stylus" scope>
  ::-webkit-scrollbar
    width 10px

  ::-webkit-scrollbar-button:start
    background-color yellow
    height 25px

  ::-webkit-scrollbar-button:end
    background-color yellow
    height 25px

  ::-webkit-scrollbar-track
    background-color orange

  ::-webkit-scrollbar-thumb
    background-color teal

  ::-webkit-scrollbar-corner
    background-color purple

  .date, .time
    text-align center
    font-size larger
    text-shadow 1px 1px 2px rgba(255, 255, 255, .5)
    font-family: 'Saira', sans-serif;
</style>
