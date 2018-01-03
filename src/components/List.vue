<template>
  <div class="container">
    <!-- Row For Buttons -->
    <div class="row justify-center" style="padding-top: 20px;">
      <q-btn color="secondary" glossy @click="showAll">All Items</q-btn>
      <q-btn color="secondary" glossy @click="showCompleted">Completed</q-btn>
      <q-btn color="secondary" glossy @click="showIncomplete">Incomplete</q-btn>
    </div>

    <!-- Input Field -->
    <div class="layout-padding" style="margin-top: 25px;">
      <q-input v-model="item.label" float-label="Add an Item" clearable @keyup.enter="addItem" :error="isError" @blur="reset" />
    </div>

    <!-- Expandable Button -->
    <q-fab class="fixed" style="right: 18px; top: 120px;" color="secondary" glossy icon="keyboard_arrow_left" direction="left">
      <q-fab-action color="secondary" icon="alarm" @click="showTime('bottom')" />
      <q-fab-action color="secondary" icon="note" @click="getNotes()" />
    </q-fab>


    <!-- Items List -->
    <q-list no-border>
      <q-list-header>My List:</q-list-header>

      <q-transition appear group enter="slideInRight" leave="slideOutLeft">

        <q-item v-for="(item, index) in items" :key="index" @click="toggleCompleted(index)" :class="{striped: isOdd(index)}">
          <!-- <q-item-side avatar="statics/boy-avatar.png" /> -->
          <q-item-side>
            <q-checkbox v-model="item.completed" @click.native="saveList" />
          </q-item-side>
          <q-item-main :label="item.label" />

          <!-- <q-item-side right icon="close" color="primary" @click.stop="removeItem(index)" /> -->

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
    </q-list>


    <!-- Time Modal -->
    <q-modal ref="timeModal" position="bottom" :content-css="{padding: '20px'}">
      <p>Clock</p>
      <q-btn color="green" @click="$refs.timeModal.close()">Close</q-btn>
    </q-modal>

    <!-- Calendar Modal -->
    <q-modal ref="calendarModal" minimized position="left" :content-css="{padding: '50px', width: '75%'}">
      <!-- Calendar -->
      <q-datetime color="secondary" v-model="date" type="datetime" float-label="Date and Time" />
      <q-btn color="green" @click="setDate">Close</q-btn>
    </q-modal>

    <!-- Edit Item Modal -->
    <q-modal ref="editItemModal" minimized position="right" :content-css="{padding: '25px', width: '100%'}">
      <!-- Textarea -->
      <q-input v-model="itemEdit" float-label="Edit Item" ref="edit" @keyup.enter="applyEdit" />
      <q-btn color="green" @click="applyEdit">Close</q-btn>
      <q-btn color="orange" @click="cancelEdit">Cancel</q-btn>
    </q-modal>


  </div>
</template>

<script>
  import { QInput, QList, QItem, QItemSide, QItemMain, QPopover, QChip, QItemTile, QTransition, QCheckbox, QListHeader, QSideLink, QBtn, QIcon, QFab, QFabAction, QModal, QDatetime } from 'quasar'

  const today = new Date()

  export default {
    components: {
      QInput, QList, QItem, QItemSide, QItemMain, QPopover, QChip, QItemTile, QTransition, QCheckbox, QListHeader, QSideLink, QBtn, QIcon, QFab, QFabAction, QModal, QDatetime
    },
    data: () => ({
      item: { label: '', reminder: '', completed: false },
      items: [],
      isError: false,
      today,
      currentIndex: 0,
      date: today,
      itemEdit: ''
    }),
    computed: {
      savedList () {
        return JSON.parse(localStorage.getItem('quasar-saved-list'))
      },
      currentItem () {
        return this.items[this.currentIndex]
      }
    },
    methods: {
      isOdd (index) {
        if (index % 2 === 0) {
          return true
        }
      },
      getList () {
        let savedList = this.savedList
        if (savedList) {
          this.items = savedList
        }
        else {
          this.items = []
        }
      },
      saveList () {
        localStorage.setItem('quasar-saved-list', JSON.stringify(this.items))
      },
      reset () {
        this.item = { label: '', completed: false }
        this.isError = false
      },
      addItem () {
        if (this.item.label === '') {
          this.isError = true
        }
        else {
          this.items.push(this.item)
          this.saveList()
          this.reset()
        }
        this.saveList()
      },
      selectItem (index) {
        // Assign current index to data-prop
        this.currentIndex = index
        if (this.currentItem.reminder) {
          this.date = this.currentItem.reminder
        }
        else {
          this.date = null
        }
      },
      removeItem (index) {
        this.$refs.popover[index].close()
        this.items.splice(index, 1)
        this.saveList()
      },
      editItem (index) {
        this.$refs.editItemModal.open()
        this.$refs.popover[index].close()
        // Autofocus input field
        this.$nextTick(() => this.$refs.edit.focus())
      },
      applyEdit () {
        if (this.itemEdit !== '') {
          this.currentItem.label = this.itemEdit
          this.itemEdit = ''
          this.saveList()
        }
        this.$refs.editItemModal.close()
      },
      cancelEdit () {
        this.$refs.editItemModal.close()
        this.itemEdit = ''
      },
      setReminder (index) {
        this.$nextTick(() => {
          this.$refs.popover[index].close()
          this.$refs.calendarModal.open()
        })
      },
      setDate () {
        this.$refs.calendarModal.close()
        // Attach date to specific item using index
        this.currentItem.reminder = this.date
        // Reset date data-prop
        this.date = null
        // Save list
        this.saveList()
        // Check items for reminder prop
        console.log(this.items)
      },
      getNotes () {
        alert('Get Notes')
      },
      toggleCompleted (index) {
        let clickedItem = this.items[index]
        clickedItem.completed ? clickedItem.completed = false : clickedItem.completed = true
        this.saveList()
      },
      showAll () {
        this.items = this.savedList
      },
      showCompleted () {
        let completed = []
        this.savedList.filter(function (item) {
          if (item.completed) {
            completed.push(item)
          }
        })
        this.items = completed
      },
      showIncomplete () {
        let incomplete = []
        this.savedList.filter(function (item) {
          if (!item.completed) {
            incomplete.push(item)
          }
        })
        this.items = incomplete
      },
      showTime (position) {
        this.position = position
        this.$nextTick(() => {
          this.$refs.timeModal.open()
        })
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

  @media all and (min-width: 600px)
    .q-btn
      margin 5px

  .striped
    background-color #f5f5f5
</style>
