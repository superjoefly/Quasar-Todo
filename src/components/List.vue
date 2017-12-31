<template>
  <div class="layout-padding">
    <div class="layout-padding" style="margin-top: -40px">
      <q-input v-model="item.label" float-label="Add an Item" clearable @keyup.enter="addItem" :error="isError" @blur="reset" />
    </div>




    <q-list no-border>
      <q-list-header>My List:</q-list-header>
      <q-transition appear group enter="fadeIn" leave="fadeOut">
        <q-item v-for="(item, index) in items" :key="index" highlight @click="toggleCompleted(index)" style="border-bottom: 1px solid #f5f3f5">
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


  </div>
</template>

<script>
  import { QInput, QList, QItem, QItemSide, QItemMain, QPopover, QChip, QItemTile, QTransition, QCheckbox, QListHeader, QSideLink } from 'quasar'
  export default {
    components: {
      QInput, QList, QItem, QItemSide, QItemMain, QPopover, QChip, QItemTile, QTransition, QCheckbox, QListHeader, QSideLink
    },
    data: () => ({
      item: { label: '', completed: false },
      items: [],
      isError: false
    }),
    methods: {
      getList () {
        let storedItems = JSON.parse(localStorage.getItem('quasar-saved-list'))
        if (storedItems) {
          this.items = storedItems
        }
        else {
          this.items = []
        }
      },
      saveList () {
        localStorage.setItem('quasar-saved-list', JSON.stringify(this.items))
        // console.log('Saved!')
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
      }
    },
    computed: {

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
    color: $red
</style>
