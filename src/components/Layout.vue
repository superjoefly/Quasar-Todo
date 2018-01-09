<template>
  <div>
    <q-layout ref="layout" view="lHh lpR fFr" :right-breakpoint="1100">
      <!-- Header -->
      <q-toolbar slot="header">
        <!-- <q-btn flat @click="$refs.layout.toggleLeft()">
          <q-icon name="menu" />
        </q-btn> -->
        <q-toolbar-title>
          Quasar Todo
          <span slot="subtitle">NewUp Developments</span>
        </q-toolbar-title>
        <!-- <q-btn flat @click="$refs.layout.toggleRight()">
          <q-icon name="menu" />
        </q-btn> -->
      </q-toolbar>

      <!-- <q-tabs slot="navigation">
        <q-route-tab slot="title" icon="list" to="/" replace hide="label" label="ALL" />
        <q-route-tab slot="title" icon="done_all" to="/completed" replace hide="label" label="COMPLETED" />
        <q-route-tab slot="title" icon="error_outline" to="/incomplete" replace hide="label" label="INCOMPLETE" />
      </q-tabs> -->


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
      <q-modal ref="timeModal" position="bottom" :content-css="{padding: '20px'}">
        <p>Clock</p>
        <q-btn color="green" @click="$refs.timeModal.close()">Close</q-btn>
      </q-modal>

      <!-- Footer -->
      <q-toolbar slot="footer" style="padding: 0px;">
        <q-search inverted v-model="query" style="box-shadow: none;" @input="searchItems" />
      </q-toolbar>
      <!-- <q-toolbar slot="footer">
        <q-toolbar-title>
          NewUp Developments
        </q-toolbar-title>
      </q-toolbar> -->
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
      memo: ''
    }),
    methods: {
      openMemo () {
        this.memo = localStorage.getItem('q-saved-memo')
        this.$refs.memoModal.open()
      },
      saveMemo () {
        localStorage.setItem('q-saved-memo', this.memo)
        this.$refs.memoModal.close()
      },
      showTime () {
        this.$nextTick(() => {
          this.$refs.timeModal.open()
        })
      },
      searchItems () {
        EventBus.$emit('searching', this.query)
      }
    },
    computed: {

    }
  }
</script>

<style lang="stylus" scope>
  ::-webkit-scrollbar
    width 10px

  ::-webkit-scrollbar-button:start
    background-color yellow
    height 25px
    // border-radius 10px 10px 0 0

  ::-webkit-scrollbar-button:end
    background-color yellow
    height 25px
    // border-radius 0px 0px 10px 10px

  // ::-webkit-scrollbar-button:horizontal:start
  //   background-color orange
  //   border-radius 10px 0 0 10px
  //
  // ::-webkit-scrollbar-button:horizontal:end
  //   background-color orange
  //   border-radius 0 10px 10px 0

  ::-webkit-scrollbar-track
    background-color orange

  ::-webkit-scrollbar-thumb
    background-color teal

  // ::-webkit-scrollbar-thumb:horizontal
  //   background-color lime

  ::-webkit-scrollbar-corner
    background-color purple


  /*Unused*/
  /*::-webkit-scrollbar-track-piece {
    background-color: brown;
  }*/
  /*::-webkit-resizer {
    background-color: red;
  }*/
</style>
