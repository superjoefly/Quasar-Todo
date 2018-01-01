<template>
  <div>
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
      <q-fab-action color="secondary" icon="date_range" @click="showCalendar" />
    </q-fab>



    <!-- Items List -->
    <q-list no-border>
      <q-list-header>My List:</q-list-header>
      <q-transition appear group enter="fadeInRight" leave="fadeOutLeft">
        <q-item v-for="(item, index) in items" :key="index" @click="toggleCompleted(index)" style="border-bottom: 1px solid #f5f3f5;">
          <!-- <q-item-side avatar="statics/boy-avatar.png" /> -->
          <q-item-side>
            <q-checkbox v-model="item.completed" @click.native="saveList" />
          </q-item-side>
          <q-item-main :label="item.label" />

          <q-item-side right icon="close" color="primary" @click.stop="removeItem(index)" />

          <!-- <q-item-side right icon="more_vert">
            <q-popover ref="popover">
              <q-list link>
                <q-item @click="removeItem(index)">
                  <q-item-main label="Delete" />
                </q-item>
                <q-item @click="$refs.popover.close()">
                  <q-item-main label="Forward" />
                </q-item>
                <q-item @click="$refs.popover.close()">
                  <q-item-main label="Delete" />
                </q-item>
              </q-list>
            </q-popover>
          </q-item-side> -->

        </q-item>
      </q-transition>
    </q-list>

    <!-- Time Modal -->
    <q-modal ref="timeModal" :position="position" :content-css="{padding: '20px'}">
      <p>Clock</p>
      <q-btn color="green" @click="$refs.timeModal.close()">Close</q-btn>
    </q-modal>

    <!-- Calendar Modal -->
    <q-modal ref="calendarModal" minimized :content-css="{padding: '50px'}">
      <p>Calendar</p>
      <q-btn color="green" @click="$refs.calendarModal.close()">Close</q-btn>
    </q-modal>


  </div>
</template>

<script>
  import { QInput, QList, QItem, QItemSide, QItemMain, QPopover, QChip, QItemTile, QTransition, QCheckbox, QListHeader, QSideLink, QBtn, QIcon, QFab, QFabAction, QModal } from 'quasar'

  export default {
    components: {
      QInput, QList, QItem, QItemSide, QItemMain, QPopover, QChip, QItemTile, QTransition, QCheckbox, QListHeader, QSideLink, QBtn, QIcon, QFab, QFabAction, QModal
    },
    data: () => ({
      item: { label: '', completed: false },
      items: [],
      isError: false,
      position: 'bottom'
    }),
    computed: {
      savedList () {
        return JSON.parse(localStorage.getItem('quasar-saved-list'))
      }
    },
    methods: {
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
        // console.log('Saved!')
        // UPDATE LIST
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
      removeItem (index) {
        this.items.splice(index, 1)
        this.saveList()
      },
      toggleCompleted (index) {
        let clickedItem = this.items[index]
        clickedItem.completed ? clickedItem.completed = false : clickedItem.completed = true
        this.saveList()
      },
      showAll () {
        this.items = this.savedList
        this.saveList()
      },
      showCompleted () {
        let completed = []
        this.savedList.filter(function (item) {
          if (item.completed) {
            completed.push(item)
          }
        })
        this.items = completed
        this.saveList()
      },
      showIncomplete () {
        let incomplete = []
        this.savedList.filter(function (item) {
          if (!item.completed) {
            incomplete.push(item)
          }
        })
        this.items = incomplete
        this.saveList()
      },
      showTime (position) {
        this.position = position
        this.$nextTick(() => {
          this.$refs.timeModal.open()
        })
      },
      showCalendar () {
        this.$nextTick(() => {
          this.$refs.calendarModal.open()
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

  .q-item-icon:hover
    color $red

  @media all and (min-width: 600px)
    .q-btn
      margin 5px
</style>
