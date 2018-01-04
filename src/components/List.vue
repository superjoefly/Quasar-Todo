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
        <q-input v-model="item.label" float-label="Add an Item" clearable @keyup.enter="addItem" :error="isError" @blur="reset" />
      </div>


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
  </q-pull-to-refresh>
</template>

<script>
  import { QInput, QList, QItem, QItemSide, QItemMain, QPopover, QChip, QItemTile, QTransition, QCheckbox, QListHeader, QSideLink, QBtn, QIcon, QFab, QFabAction, QModal, QDatetime, QPullToRefresh, QFixedPosition, Toast } from 'quasar'

  export default {
    components: {
      QInput, QList, QItem, QItemSide, QItemMain, QPopover, QChip, QItemTile, QTransition, QCheckbox, QListHeader, QSideLink, QBtn, QIcon, QFab, QFabAction, QModal, QDatetime, QPullToRefresh, QFixedPosition
    },
    data: () => ({
      item: { label: '', reminder: '', notes: '', completed: false },
      items: [],
      isError: false,
      date: new Date(),
      itemEdit: '',
      selectedItem: {}
    }),
    computed: {
    },
    methods: {
      isOdd (index) {
        if (index % 2 === 0) {
          return true
        }
      },
      savedList () {
        return JSON.parse(localStorage.getItem('q-saved-list'))
      },
      getList () {
        let savedList = this.savedList()
        if (savedList) {
          this.items = savedList
        }
        else {
          this.items = []
        }
      },
      saveList () {
        localStorage.setItem('q-saved-list', JSON.stringify(this.items))
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
      },
      selectItem (index) {
        this.selectedItem = this.items[index]
        console.log(this.selectedItem)
        if (this.selectedItem.reminder) {
          this.date = this.selectedItem.reminder
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
          this.selectedItem.label = this.itemEdit
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
        // Attach date to selected item using index
        this.selectedItem.reminder = this.date
        // Reset date data-prop
        this.date = null
        // Save list
        this.saveList()
      },
      toggleCompleted (index) {
        let clickedItem = this.items[index]
        clickedItem.completed ? clickedItem.completed = false : clickedItem.completed = true
        this.saveList()
      },
      showAll () {
        this.items = this.savedList()
      },
      showCompleted () {
        let completedItems = []
        this.savedList().filter(function (item) {
          if (item.completed) {
            completedItems.push(item)
          }
        })
        this.items = completedItems
      },
      showIncomplete () {
        let incompleteItems = []
        this.savedList().filter(function (item) {
          if (!item.completed) {
            incompleteItems.push(item)
          }
        })
        this.items = incompleteItems
      },
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

</style>
