<template>
  <q-pull-to-refresh :handler="refresh" refresh-message="Updating List...">
    <div class="container layout-padding">
      <!-- Buttons -->
      <div class="row justify-center">
        <q-btn color="secondary" glossy @click="itemView = 'All'; selection = 'A'" :class="{active: isActive('A')}">
          <q-icon name="list" />
          &nbsp;
          <span class="gt-sm">All Items</span>
        </q-btn>
        <q-btn color="secondary" glossy @click="itemView = 'Completed'; selection = 'B'" :class="{active: isActive('B')}">
          <q-icon name="done_all" />
          &nbsp;
          <span class="gt-sm">Completed</span>
        </q-btn>
        <q-btn color="secondary" glossy @click="itemView = 'Incomplete'; selection = 'C'" :class="{active: isActive('C')}">
          <q-icon name="error_outline" />
          &nbsp;
          <span class="gt-sm">Incomplete</span>
        </q-btn>
        <q-btn color="secondary" glossy @click="itemView = 'Starred'; selection = 'D'" :class="{active: isActive('D')}">
          <q-icon name="star_border" />
          &nbsp;
          <span class="gt-sm">Starred</span>
        </q-btn>
      </div>

      <!-- Input Field -->
      <div class="layout-padding">
        <q-input v-model="item.label" float-label="Add an Item..." clearable @keyup.enter="addItem" :error="isError" @blur="resetItem"></q-input>
      </div>


      <q-list no-border style="margin-bottom: 25px;">
        <q-list-header>{{ itemView }}</q-list-header>
        <q-transition group appear enter="fadeIn"
        leave="fadeOut">
          <q-item v-for="(item, index) in computedItems" :key="index" @click="toggleCompleted(index)"
          :class="{striped: isOdd(index)}" :data-index="index">

            <q-item-side>
              <q-checkbox v-model="item.completed" @click.native="saveList()" />
            </q-item-side>

            <q-item-main :label="item.label" :class="{strike: item.completed}" />

            <q-checkbox v-model="item.starred" checked-icon="star" unchecked-icon="star" @click.native="saveList" color="yellow" />

            <q-item-side right icon="more_vert" @click="selectItem(index)">

              <q-popover ref="popover">
                <q-list link separator>
                  <q-item @click="removeItem(index)">
                    <q-item-main label="Delete" />
                  </q-item>
                  <q-item @click="editItem(index)">
                    <q-item-main label="Edit" />
                  </q-item>
                  <q-item @click="openReminder(index)">
                    <q-item-main label="Reminder" />
                  </q-item>
                  <q-item @click="addNotes(index)">
                    <q-item-main label="Notes" />
                  </q-item>
                </q-list>
              </q-popover>
            </q-item-side>

            <q-context-menu ref="popover">
              <q-list link separator>
                <q-item @click="removeItem(index)">
                  <q-item-main label="Delete" />
                </q-item>
                <q-item @click="editItem(index)">
                  <q-item-main label="Edit" />
                </q-item>
                <q-item @click="openReminder(index)">
                  <q-item-main label="Reminder" />
                </q-item>
                <q-item @click="addNotes(index)">
                  <q-item-main label="Notes" />
                </q-item>
              </q-list>
            </q-context-menu>
          </q-item>
        </q-transition>
      </q-list>


      <!-- Edit Item Modal -->
      <q-modal ref="editItemModal" minimized position="right" :content-css="{padding: '25px', width: '100%'}" class="blue-backdrop">
        <!-- Textarea -->
        <q-input v-model="selectedItem.label" float-label="Edit Item" ref="edit" @keyup.enter="saveEdit" />
        <q-btn color="green" @click="saveEdit">Save</q-btn>
      </q-modal>

      <q-modal ref="calendarModal" minimized position="left" :content-css="{padding: '50px', width: 'auto'}">
        <!-- Calendar -->
        <q-inline-datetime color="secondary" v-model="currentDate" type="datetime" @click.native="setDate" />
      </q-modal>


      <!-- Calendar Modal -->
      <!-- <q-modal ref="calendarModal" minimized position="left" :content-css="{padding: '50px', width: '75%'}"> -->
        <!-- Calendar -->

        <!-- <q-datetime color="secondary" v-model="currentDate" type="datetime" float-label="Set Reminder" />
        <q-btn color="green" @click="setDate">Save</q-btn>
      </q-modal> -->


      <!-- Item Notes Modal -->
      <q-modal ref="notesModal" minimized :content-css="{padding: '25px'}" class="yellow-backdrop">
        <!-- Textarea -->
        <q-input v-model="selectedItem.notes" type="textarea" :float-label="selectedItem.label" :max-height="100" :min-rows="7" ref="notes" />
        <q-btn color="yellow-8" @click="saveNotes">Save</q-btn>
      </q-modal>

    </div>
  </q-pull-to-refresh>
</template>

<script>
/*eslint-disable*/


import { EventBus } from '../main.js'

import { QInput, QList, QItem, QItemSide, QItemMain, QPopover, QChip, QItemTile, QTransition, QCheckbox, QListHeader, QSideLink, QBtn, QIcon, QFab, QFabAction, QModal, QPullToRefresh, QFixedPosition, Toast, QSearch, date, QContextMenu, QInlineDatetime } from 'quasar'

export default {
  components: {
    QInput, QList, QItem, QItemSide, QItemMain, QPopover, QChip, QItemTile, QTransition, QCheckbox, QListHeader, QSideLink, QBtn, QIcon, QFab, QFabAction, QModal, QPullToRefresh, QFixedPosition, QSearch, QContextMenu, QInlineDatetime
  },
  data () {
    return {
      item: { label: '', reminder: '', notes: '', completed: false, starred: false },
      masterList: [],
      isError: false,
      currentDate: Date.now(),
      selectedItem: {},
      itemView: 'All',
      query: '',
      selection: 'A'
    }
  },
  computed: {
    itemReminder () {
      return this.selectedItem.reminder ? this.selectedItem.reminder : this.currentDate
    },
    computedItems: function () {
      var vm = this
      let view = vm.itemView
      return this.masterList.filter(function (item) {
        if (view === 'Completed') {
          return item.completed
        }
        else if (view === 'Incomplete') {
          return !item.completed
        }
        else if (view === 'Starred') {
          return item.starred
        }
        else if (view === 'search') {
          return item.label.toLowerCase().indexOf(vm.query.toLowerCase()) !== -1
        }
        else {
          return item
        }
      })
    }
  },
  methods: {
    // onDeviceReady() {
    //     // console.log(navigator.notification);
    //
    //     console.log("Hello")
    //
    //     // navigator.notification.alert("The time is now...", alertCallback, [title], [buttonName])
    // },
    savedList () {
      return JSON.parse(localStorage.getItem('q-saved-list'))
    },
    getList () {
      if (this.savedList()) {
        this.masterList = this.savedList()
      }
      else {
        this.masterList = []
      }
    },
    addItem () {
      if (this.item.label === '') {
        this.isError = true
      }
      else {
        this.masterList.push(this.item)
        this.saveList()
        this.resetItem()
      }
    },
    resetItem () {
      this.item = { label: '', reminder: '', notes: '', completed: false, starred: false }
      this.isError = false
    },
    saveList () {
      localStorage.setItem('q-saved-list', JSON.stringify(this.masterList))
    },
    selectItem (index) {
      this.selectedItem = this.computedItems[index]
      if (this.selectedItem.reminder) {
        this.currentDate = this.selectedItem.reminder
      }
      else {
        this.currentDate = new Date()
      }
    },
    removeItem (index) {
      this.masterList.splice(index, 1)
      this.$refs.popover[index].close()
      this.saveList()
    },
    editItem (index) {
      this.$refs.editItemModal.open()
      this.$refs.popover[index].close()
      // Autofocus Input Field
      this.$nextTick(() => this.$refs.edit.focus())
    },
    saveEdit () {
      this.saveList()
      this.$refs.editItemModal.close()
    },
    openReminder (index) {
      this.$refs.calendarModal.open()
      this.$refs.popover[index].close()
    },
    formatDate (dateSet) {
      return date.formatDate(dateSet, 'ddd Do, YYYY @ H:mm a')
    },
    setDate () {
      this.$refs.calendarModal.close()
      this.selectedItem.reminder = this.currentDate

      let formattedDate = this.formatDate(this.currentDate)

      Toast.create({
        html: `Reminder set for <span style="color: teal;">${formattedDate}</span>`,
        icon: 'check',
        timeout: 3000,
        color: 'bg-secondary',
        bgColor: '#eaffe8'
      })
      this.currentDate = Date.now()
      this.saveList()
    },
    addNotes (index) {
      this.$refs.notesModal.open()
      this.$refs.popover[index].close()
      this.$nextTick(() => this.$refs.notes.focus())
    },
    saveNotes () {
      this.$refs.notesModal.close()
      this.saveList()
    },
    toggleCompleted (index) {
      let clickedItem = this.masterList[index]
      clickedItem.completed ? clickedItem.completed = false : clickedItem.completed = true
      this.saveList()
    },
    refresh (done) {
      setTimeout(() => {
        done()
        // this.saveList()
        this.getList()
        Toast.create({
          html: 'Up To Date!',
          icon: 'check',
          timeout: 3000,
          color: 'green',
          bgColor: '#eaffe8'
        })
      }, 1000)
    },
    isOdd (index) {
      // Returns true if index / 2 = 0
      return index % 2 === 0
    },
    isActive (value) {
      // Returns true when selection and value are equal
      return this.selection === value
    },
    sayHi() {
      console.log("Hello")
    },
    alertCallback () {
      this.getList()
    },
    showNotification () {
      cordova.plugins.notification.local.schedule({
        title: 'Well, hello there!',
        text: 'That was easy!',
        foreground: true
      })
    }
  },
  created () {
    // Register the event:
    document.addEventListener("deviceready", this.showNotification, false);

    this.getList()
    EventBus.$on('searching', (query) => {
      this.query = query
      this.itemView = 'search'
    })
  },
  beforeDestroy () {
    // Remove eventListener:
    document.removeEventListener('deviceready', this.showNotification, false)
  }
}
</script>

<style lang="stylus" scoped>
  @import '~variables'

  .container
    max-width 100%
    overflow: hidden

  .q-btn
      margin 0px 10px 0px 10px

  .striped
    background-color #f5f5f5

  .yellow-backdrop
    background-color rgba(255, 255, 184, .4)

  .blue-backdrop
    background-color rgba(233, 233, 255, .4)

  .active
    color #FFFFA1 !important

  .strike
    text-decoration: line-through
</style>
