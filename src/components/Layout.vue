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
        <q-search inverted v-model="search" style="box-shadow: none;" />
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
  import { QLayout, QToolbar, QToolbarTitle, QBtn, QIcon, QTabs, QRouteTab, QList, QListHeader, QSideLink, QItemSide, QItemMain, QSearch, QFixedPosition, QFab, QFabAction, QModal, QInput } from 'quasar'
  export default {
    components: {
      QLayout, QToolbar, QToolbarTitle, QBtn, QIcon, QTabs, QRouteTab, QList, QListHeader, QSideLink, QItemSide, QItemMain, QSearch, QFixedPosition, QFab, QFabAction, QModal, QInput
    },
    data: () => ({
      search: '',
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
      }
    },
    computed: {

    }
  }
</script>

<style lang="stylus" scoped>

</style>
