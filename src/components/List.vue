<template>
  <q-pull-to-refresh :handler="refresh" refresh-message="Updating...">
    <div class="container layout-padding">
      <!-- Row For Buttons -->
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
        <q-input v-model="item.label" float-label="Add an Item" clearable @keyup.enter="addItem" :error="isError" @blur="resetItem" />
      </div>


      <!-- Items List -->
      <!-- <q-list no-border>
        <q-list-header>My List:</q-list-header>

        <q-transition appear group enter="slideInRight" leave="slideOutLeft">
          <q-item v-for="(item, index) in items" :key="index" @click="toggleCompleted(index)" :class="{striped: isOdd(index)}">
            <q-item-side avatar="statics/boy-avatar.png" />
            <q-item-side>
              <q-checkbox v-model="item.completed" @click.native="saveList" />
            </q-item-side>
            <q-item-main :label="item.label" />

            <q-item-side right icon="close" color="primary" @click.stop="removeItem(index)" />

            <q-item-side right icon="more_vert" @click="selectItem(index)">
              <q-popover ref="popover">
                <q-list link separator>
                  <q-item @click="removeItem(index)">
                    <q-item-main label="Delete" />
                  </q-item>
                  <q-item @click="editItem(index)">
                    <q-item-main label="Edit" />
                  </q-item>
                  <q-item @click="setReminder(index)">
                    <q-item-main label="Reminder" />
                  </q-item>
                </q-list>
              </q-popover>
            </q-item-side>

          </q-item>
        </q-transition>
      </q-list> -->


      <q-list no-border>
        <q-list-header>My List:</q-list-header>
        <transition-group @before-enter="beforeEnter" @enter="enter" @leave="leave">
          <q-item v-for="(item, index) in items" :key="index" @click="toggleCompleted(index)" :class="{striped: isOdd(index)}" v-bind:data-index="index">

            <!-- <q-item-side avatar="statics/boy-avatar.png" /> -->
            <q-item-side>
              <q-checkbox v-model="item.completed" @click.native="saveList" />
            </q-item-side>
            <q-item-main :label="item.label" />

            <!-- <q-item-side right icon="close" color="primary" @click.stop="removeItem(index)" /> -->

              <q-checkbox v-model="item.starred" checked-icon="star" unchecked-icon="star" @click.native="saveList" :key="index" color="yellow" />

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
  import { QInput, QList, QItem, QItemSide, QItemMain, QPopover, QChip, QItemTile, QTransition, QCheckbox, QListHeader, QSideLink, QBtn, QIcon, QFab, QFabAction, QModal, QDatetime, QPullToRefresh, QFixedPosition, Toast } from 'quasar'

  export default {
    components: {
      QInput, QList, QItem, QItemSide, QItemMain, QPopover, QChip, QItemTile, QTransition, QCheckbox, QListHeader, QSideLink, QBtn, QIcon, QFab, QFabAction, QModal, QDatetime, QPullToRefresh, QFixedPosition
    },
    data: () => ({
      item: { label: '', reminder: '', notes: '', completed: false, starred: false },
      items: [],
      isError: false,
      date: new Date(),
      selectedItem: {}
    }),
    computed: {
      itemReminder () {
        return this.selectedItem.reminder ? this.selectedItem.reminder : this.date
      }
    },
    methods: {
      // For styling odd list-items
      isOdd (index) {
        if (index % 2 === 0) {
          return true
        }
      },
      // Retrieve saved list from localStorage
      savedList () {
        return JSON.parse(localStorage.getItem('q-saved-list'))
      },
      // Get and display list-items
      getList () {
        let savedList = this.savedList()
        if (savedList) {
          this.items = savedList
        }
        else {
          this.items = []
        }
      },
      // Save current list
      saveList () {
        localStorage.setItem('q-saved-list', JSON.stringify(this.items))
      },
      // Reset item
      resetItem () {
        this.item = { label: '', reminder: '', notes: '', completed: false, starred: false }
        this.isError = false
      },
      // Add item to current list
      addItem () {
        if (this.item.label === '') {
          this.isError = true
        }
        else {
          this.items.push(this.item)
          this.saveList()
          this.resetItem()
        }
      },
      // Select current item
      selectItem (index) {
        this.selectedItem = this.items[index]
        console.log(this.selectedItem)
        if (this.selectedItem.reminder) {
          this.date = this.selectedItem.reminder
        }
        else {
          // return
          this.date = new Date()
        }
      },
      // Open calendar
      openReminder (index) {
        this.$nextTick(() => {
          this.$refs.popover[index].close()
          this.$refs.calendarModal.open()
        })
      },
      // Set date for reminder
      setDate () {
        // Close modal
        this.$refs.calendarModal.close()
        // Attach date to selected item
        this.selectedItem.reminder = this.date
        // Reset date data-prop
        this.date = null
        // Save list
        this.saveList()
      },
      // Delete item
      removeItem (index) {
        this.$refs.popover[index].close()
        this.items.splice(index, 1)
        this.saveList()
      },
      // Edit item label
      editItem (index) {
        this.$refs.editItemModal.open()
        this.$refs.popover[index].close()
        // Autofocus input field
        this.$nextTick(() => this.$refs.edit.focus())
      },
      // Save item label
      saveEdit () {
        this.saveList()
        this.$refs.editItemModal.close()
      },
      // Add item note
      addNotes (index) {
        this.$refs.notesModal.open()
        this.$refs.popover[index].close()
        // Autofocus input field
        this.$nextTick(() => this.$refs.notes.focus())
      },
      // Save item note
      saveNotes () {
        this.saveList()
        this.$refs.notesModal.close()
      },
      // Toggle item completed
      toggleCompleted (index) {
        let clickedItem = this.items[index]
        clickedItem.completed ? clickedItem.completed = false : clickedItem.completed = true
        this.saveList()
      },
      toggleStared (index) {
        let clickedItem = this.items[index]
        clickedItem.starred ? clickedItem.starred = false : clickedItem.starred = true
        this.saveList()
      },
      // Show all items
      showAll () {
        this.items = this.savedList()
      },
      // Show only completed items
      showCompleted () {
        let completedItems = []
        this.savedList().filter(function (item) {
          if (item.completed) {
            completedItems.push(item)
          }
        })
        this.items = completedItems
      },
      // Show only incomplete items
      showIncomplete () {
        let incompleteItems = []
        this.savedList().filter(function (item) {
          if (!item.completed) {
            incompleteItems.push(item)
          }
        })
        this.items = incompleteItems
      },
      // Refresh list - show all items
      refresh (done) {
        setTimeout(() => {
          done()
          this.saveList()
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
          el.classList.add('animated', 'bounceInRight')
        }, delay)
      },
      leave (el, done) {
        var delay = el.dataset.index * 150
        setTimeout(function () {
          el.classList.add('animated', 'bounceOutLeft')
        }, delay)
      }
    },
    // Get saved list items
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
