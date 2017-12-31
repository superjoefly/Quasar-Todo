<template>
  <div>
    <q-layout ref="layout" view="hHr LpR lFf" :right-breakpoint="1100">
      <!-- Header -->
      <q-toolbar slot="header">
        <!-- <q-btn flat @click="$refs.layout.toggleLeft()">
          <q-icon name="menu" />
        </q-btn> -->
        <q-toolbar-title>
          Quasar Todo
          <span slot="subtitle">Optional subtitle</span>
        </q-toolbar-title>
        <!-- <q-btn flat @click="$refs.layout.toggleRight()">
          <q-icon name="menu" />
        </q-btn> -->
      </q-toolbar>
      <!-- Navigation -->
      <q-tabs slot="navigation">
        <q-route-tab slot="title" icon="list" to="/list" replace hide="icon" label="List" />
        <q-route-tab slot="title" icon="!" to="/incomplete" replace hide="icon" label="Incomplete" />
        <q-route-tab slot="title" icon="check_circle" to="/complete" replace label="Completed" />
        <!-- <q-route-tab slot="title" icon="input" to="/" replace label="Drawer" /> -->
      </q-tabs>
      <!-- Left Side Panel -->
      <!-- <div slot="left">
        <q-list no-border link inset-separator>
          <q-list-header>Essential Links</q-list-header>
          <q-side-link item to="/docs">
            <q-item-side icon="school" />
            <q-item-main label="Docs" sublabel="quasar-framework.org" />
          </q-side-link>
          <q-side-link item to="/forum">
            <q-item-side icon="record_voice_over" />
            <q-item-main label="Forum" sublabel="forum.quasar-framework.org" />
          </q-side-link>
          <q-side-link item to="/chat">
            <q-item-side icon="chat" />
            <q-item-main label="Gitter Channel" sublabel="Quasar Lobby" />
          </q-side-link>
          <q-side-link item to="/twitter">
            <q-item-side icon="rss feed" />
            <q-item-main label="Twitter" sublabel="@quasarframework" />
          </q-side-link>
        </q-list>
      </div> -->
      <!-- Right Side Panel -->
      <!-- <div slot="right">
        Right Side of Layout
      </div> -->
      <!-- sub-routes get injected here: -->
      <router-view />
      <!-- Footer -->
      <q-toolbar slot="footer">
        <q-toolbar-title>
          NewUp Developments
        </q-toolbar-title>
      </q-toolbar>
    </q-layout>

  </div>
</template>

<script>
  import { QLayout, QToolbar, QToolbarTitle, QBtn, QIcon, QTabs, QRouteTab, QList, QListHeader, QSideLink, QItemSide, QItemMain } from 'quasar'
  export default {
    components: {
      QLayout, QToolbar, QToolbarTitle, QBtn, QIcon, QTabs, QRouteTab, QList, QListHeader, QSideLink, QItemSide, QItemMain
    },
    data: () => ({

    }),
    methods: {

    },
    computed: {

    }
  }
</script>

<style lang="stylus" scoped>

</style>
