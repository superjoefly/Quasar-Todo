<template>
  <q-pull-to-refresh :handler="refresh" refresh-message="Updating List...">
    <div class="container layout-padding">
      <!-- Buttons -->
      <div class="row justify-center">
        <q-btn color="secondary" glossy @click="itemView = 'all'">
          <q-icon name="list" />
          &nbsp;
          <span class="gt-sm">All Items</span>
        </q-btn>
        <q-btn color="secondary" glossy @click="itemView = 'completed'">
          <q-icon name="done_all" />
          &nbsp;
          <span class="gt-sm">Completed</span>
        </q-btn>
        <q-btn color="secondary" glossy @click="itemView = 'incomplete'">
          <q-icon name="error_outline" />
          &nbsp;
          <span class="gt-sm">Incomplete</span>
        </q-btn>
      </div>

      <!-- Input Field -->
      <div class="layout-padding">
        <q-input v-model="item.label" float-label="Add an Item..." clearable @keyup.enter="addItem" :error="isError" @blur="resetItem"></q-input>
      </div>

      <!-- Master List -->
      <!-- <q-list no-border>
        <q-list-header>All Items:</q-list-header>
        <transition-group @before-enter="beforeEnter" @enter="enter" @leave="leave" appear mode="out-in" :css="false">
          <q-item v-for="(item, index) in computedItems" :key="index" @click="toggleCompleted(index)" :class="{striped: isOdd(index)}" :data-index="index">

            <q-item-side>
              <q-checkbox v-model="item.completed" @click.native="saveList" />
            </q-item-side>
            <q-item-main :label="item.label" />

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
          </q-item>
        </transition-group>
      </q-list> -->


      <q-list no-border style="margin-bottom: 25px;">
        <q-list-header>All Items:</q-list-header>
        <q-transition group appear enter="fadeIn"
        leave="fadeOut">
          <q-item v-for="(item, index) in computedItems" :key="index" @click="toggleCompleted(index)" :class="{striped: isOdd(index)}" :data-index="index">

            <q-item-side>
              <q-checkbox v-model="item.completed" @click.native="saveList" />
            </q-item-side>
            <q-item-main :label="item.label" />

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
          </q-item>
        </q-transition>
      </q-list>



      <!-- Edit Item Modal -->
      <q-modal ref="editItemModal" minimized position="right" :content-css="{padding: '25px', width: '100%'}" class="blue-backdrop">
        <!-- Textarea -->
        <q-input v-model="selectedItem.label" float-label="Edit Item" ref="edit" @keyup.enter="saveEdit" />
        <q-btn color="green" @click="saveEdit">Save</q-btn>
      </q-modal>

      <!-- Calendar Modal -->
      <q-modal ref="calendarModal" minimized position="left" :content-css="{padding: '50px', width: '75%'}">
        <!-- Calendar -->
        <q-datetime color="secondary" v-model="date" type="datetime" float-label="Set Reminer" />
        <q-btn color="green" @click="setDate">Save</q-btn>
      </q-modal>


      <!-- Item Notes Modal -->
      <q-modal ref="notesModal" minimized :content-css="{padding: '25px'}" class="yellow-backdrop">
        <!-- Textarea -->
        <q-input v-model="selectedItem.notes" type="textarea" :float-label="selectedItem.label" :max-height="100" :min-rows="7" ref="notes" />
        <q-btn color="yellow-8" @click="saveNotes">Save</q-btn>
      </q-modal>

      <!-- <p>{{computedItems}}</p> -->

    </div>
  </q-pull-to-refresh>
</template>

<script>
/*eslint-disable*/

import { EventBus } from '../main.js'

import { QInput, QList, QItem, QItemSide, QItemMain, QPopover, QChip, QItemTile, QTransition, QCheckbox, QListHeader, QSideLink, QBtn, QIcon, QFab, QFabAction, QModal, QDatetime, QPullToRefresh, QFixedPosition, Toast, QSearch } from 'quasar'

export default {
  components: {
    QInput, QList, QItem, QItemSide, QItemMain, QPopover, QChip, QItemTile, QTransition, QCheckbox, QListHeader, QSideLink, QBtn, QIcon, QFab, QFabAction, QModal, QDatetime, QPullToRefresh, QFixedPosition, QSearch
  },
  data () {
    return {
      item: { label: '', reminder: '', notes: '', completed: false, starred: false },
      masterList: [],
      isError: false,
      date: new Date(),
      selectedItem: {},
      itemView: 'all',
      query: ''
    }
  },
  computed: {
    itemReminder () {
      return this.selectedItem.reminder ? this.selectedItem.reminder : this.date
    },
    computedItems: function () {
      var vm = this
      let view = vm.itemView
      return this.masterList.filter(function (item) {
        if (view === 'completed') {
          return item.completed
        }
        else if (view === 'incomplete') {
          return !item.completed
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
    savedList () {
      return JSON.parse(localStorage.getItem('q-saved-list'))
    },
    getList () {
      console.log('Getting List...')
      if (this.savedList()) {
        this.masterList = this.savedList()
      }
      else {
        this.masterList = []
      }
    },
    addItem () {
      console.log('Add Item!')
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
      console.log('Reset Item!')
      this.item = { label: '', reminder: '', notes: '', completed: false, starred: false }
      this.isError = false
    },
    saveList () {
      console.log('Saving!')
      localStorage.setItem('q-saved-list', JSON.stringify(this.masterList))
    },
    selectItem (index) {
      console.log('Item Selected!')
      this.selectedItem = this.computedItems[index]
      console.log(this.selectedItem)
      if (this.selectedItem.reminder) {
        this.date = this.selectedItem.reminder
      }
      else {
        this.date = new Date()
      }
      console.log(this.selectedItem)
    },
    removeItem (index) {
      console.log('Remove Item!')
      this.masterList.splice(index, 1)
      this.$refs.popover[index].close()
      this.saveList()
    },
    editItem (index) {
      console.log('Open Item Edit Modal!')
      this.$refs.editItemModal.open()
      this.$refs.popover[index].close()
      // Autofocus Input Field
      this.$nextTick(() => this.$refs.edit.focus())
    },
    saveEdit () {
      console.log('Save Item Label!')
      this.saveList()
      this.$refs.editItemModal.close()
    },
    openReminder (index) {
      console.log('Open Reminder Modal!')
      this.$refs.calendarModal.open()
      this.$refs.popover[index].close()
    },
    setDate () {
      console.log('Set Date!')
      this.$refs.calendarModal.close()
      this.selectedItem.reminder = this.date
      this.date = new Date()
      this.saveList()
    },
    addNotes (index) {
      console.log('Open Notes Modal!')
      this.$refs.notesModal.open()
      this.$refs.popover[index].close()
      this.$nextTick(() => this.$refs.notes.focus())
    },
    saveNotes () {
      console.log('Save Item Notes!')
      this.$refs.notesModal.close()
      this.saveList()
    },
    toggleCompleted (index) {
      console.log('Toggle Completed!')
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
      console.log('refreshing!')
    },
    // // List item transitions
    // beforeEnter (el) {
    //   el.style.opacity = 0
    // },
    // enter (el, done) {
    //   var delay = el.dataset.index * 150
    //   setTimeout(function () {
    //     el.style.opacity = 1
    //     el.classList.add('animated', 'bounceInRight')
    //   }, delay)
    //   done()
    // },
    // leave (el, done) {
    //   var delay = el.dataset.index * 150
    //   setTimeout(function () {
    //     el.classList.add('animated', 'bounceOutLeft')
    //     done()
    //   }, delay)
    // },
    isOdd (index) {
      if (index % 2 === 0) {
        return true
      }
    },
    searchItems () {
      console.log('Searching')
    }
  },
  created () {
    console.log('Created!')
    // localStorage.clear()
    this.getList()
    EventBus.$on('searching', (query) => {
      this.query = query
      this.itemView = 'search'
    })
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
</style>
