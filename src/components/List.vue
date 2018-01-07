<template>
  <q-pull-to-refresh :handler="refresh" refresh-message="Updating List...">
    <div class="container layout-padding">
      <!-- Buttons -->
      <div class="row justify-center">
        <q-btn color="secondary" glossy @click="showAll">
          <q-icon name="list" />
          &nbsp;
          <span class="gt-sm">All Items</span>
        </q-btn>
        <q-btn color="secondary" glossy @click="showCompleted">
          <q-icon name="done_all" />
          &nbsp;
          <span class="gt-sm">Completed</span>
        </q-btn>
        <q-btn color="secondary" glossy @click="showIncomplete">
          <q-icon name="error_outline" />
          &nbsp;
          <span class="gt-sm">Incomplete</span>
        </q-btn>
      </div>

      <!-- Input Field -->
      <div class="layout-padding">
        <q-input v-model="item.label" float-label="Add an Item..." clearable @keyup.enter="addItem" :error="isError" @blur="resetItem"></q-input>
      </div>

      <!-- User List -->
      <q-list no-border>
        <q-list-header>My List:</q-list-header>
        <transition-group @before-enter="beforeEnter" @enter="enter" @leave="leave">
          <q-item v-for="(item, index) in masterList" :key="index" @click="toggleCompleted(index)" :class="{striped: isOdd(index)}" :data-index="index">

            <q-item-side>
              <q-checkbox v-model="item.completed" @click.native="saveList"></q-checkbox>
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
      </q-list>

      <!-- Calendar Modal -->
      <q-modal ref="calendarModal" minimized position="left" :content-css="{padding: '50px', width: '75%'}">
        <!-- Calendar -->
        <q-datetime color="secondary" v-model="date" type="datetime" float-label="Set Reminer" />
        <q-btn color="green" @click="setDate">Close</q-btn>
      </q-modal>

      <!-- Edit Item Modal -->
      <q-modal ref="editItemModal" minimized position="right" :content-css="{padding: '25px', width: '100%'}" class="blue-backdrop">
        <!-- Textarea -->
        <q-input v-model="selectedItem.label" float-label="Edit Item" ref="edit" @keyup.enter="saveEdit" />
        <q-btn color="green" @click="saveEdit">Save</q-btn>
      </q-modal>

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
import { QInput, QList, QItem, QItemSide, QItemMain, QPopover, QChip, QItemTile, QTransition, QCheckbox, QListHeader, QSideLink, QBtn, QIcon, QFab, QFabAction, QModal, QDatetime, QPullToRefresh, QFixedPosition, Toast } from 'quasar'

export default {
  components: {
    QInput, QList, QItem, QItemSide, QItemMain, QPopover, QChip, QItemTile, QTransition, QCheckbox, QListHeader, QSideLink, QBtn, QIcon, QFab, QFabAction, QModal, QDatetime, QPullToRefresh, QFixedPosition
  },
  data: () => ({
    item: { label: '', reminder: '', notes: '', completed: false, starred: false },
    isError: false,
    masterList: [],
    date: new Date(),
    selectedItem: {}
  }),
  computed: {
    itemReminder () {
      return this.selectedItem.reminder ? this.selectedItem.reminder : this.date
    }
  },
  methods: {
    getList () {
      console.log('Getting List...')
    },
    addItem () {
      console.log('Add Item!')
    },
    resetItem () {
      console.log('Reset Item!')
    },
    saveList () {
      console.log('Saving!')
    },
    selectItem (index) {
      console.log('Item Selected!')
    },
    removeItem (index) {
      console.log('Remove Item!')
    },
    editItem (index) {
      console.log('Open Item Edit Modal!')
    },
    openReminder (index) {
      console.log('Open Reminder Modal!')
    },
    addNotes (index) {
      console.log('Open Notes Modal!')
    },
    setDate () {
      console.log('Set Date!')
    },
    saveEdit () {
      console.log('Save Item Label!')
    },
    saveNotes () {
      console.log('Save Item Notes!')
    },
    showAll () {
      console.log('Show All')
    },
    showCompleted () {
      console.log('Show Completed')
    },
    showIncomplete () {
      console.log('Show Incomplete')
    },
    toggleCompleted (index) {
      console.log('Toggle Completed!')
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
    // List item transitions
    beforeEnter (el) {
      el.style.opacity = 0
    },
    enter (el, done) {
      var delay = el.dataset.index * 150
      setTimeout(function () {
        el.style.opacity = 1
        el.classList.add('animated', 'fadeIn')
      }, delay)
    },
    leave (el, done) {
      var delay = el.dataset.index * 150
      setTimeout(function () {
        el.classList.add('animated', 'bounceOutLeft')
      }, delay)
    },
    isOdd (index) {
      if (index % 2 ===0) {
        return true
      }
    }
  },
  mounted () {
    // localStorage.clear()
    this.getList()
  }
}

</script>

<style lang="stylus" scoped>
  @import '~variables'

  .container
    max-width 100%
    overflow: hidden

  // .q-item-icon:hover
  //   color $orange

  .q-btn
      margin 0px 10px 0px 10px

  .striped
    background-color #f5f5f5

  .yellow-backdrop
    background-color rgba(255, 255, 184, .8)

  .blue-backdrop
    background-color rgba(233, 233, 255, .6)
</style>
