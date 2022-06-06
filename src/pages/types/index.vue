<template>
  <q-card flat>
    <q-toolbar>
      <div v-if="$route.path!=='/manager/types/view'" class="col-auto">
        <q-btn flat dense icon="arrow_back" v-close-popup />
      </div>
      <q-toolbar-title>{{$t('route.types')}} </q-toolbar-title>
      <q-btn v-if="isRoutes.add" icon="add" flat round dense color="blue" @click="onAdd" />
      <q-btn icon="filter_list" flat round dense color="teal">
        <q-tooltip>{{$t('global.filter')}}</q-tooltip>
        <q-menu v-model="isFilter" class="q-pa-md">
          <!-- <div class="q-pa-md"> -->
          <div class="row">
            <div class="col-12">
              <q-input v-model="pagination.filter" :dense="$store.getters.dense.input" debounce="500" :placeholder="$t('global.search')">
                <template v-slot:append>
                  <q-icon v-if="pagination.filter===''" name="search" />
                  <q-icon v-else name="clear" class="cursor-pointer" @click="pagination.filter=''" />
                </template>
              </q-input>
            </div>
          </div>
          <div class="row">
            <div class="col-12">
              <q-select v-model="pagination.key" :options="$store.state.types.keys" dense options-dense :label="$t('global.types')"
                        @update:model-value="isFilter=!isFilter">
                <template v-slot:selected>
                  <div v-html="`${$t(`types.${pagination.key}`)} - ${pagination.key}`"></div>
                </template>
                <template v-slot:option="scope">
                  <q-item v-bind="scope.itemProps">
                    <q-item-section>
                      <q-item-label v-html="`${$t(`types.${scope.opt}`)} - ${scope.opt}`" />
                    </q-item-section>
                  </q-item>
                </template>
              </q-select>
            </div>
          </div>
          <div class="row">
            <div class="col-12">
              <q-toggle v-model="pagination.flag" left-label :label="pagination.flag?$t('global.working'):$t('global.trash')"
                        :false-value="0" :true-value="1" @update:model-value="onChangeFlag" />
            </div>
          </div>
        </q-menu>
      </q-btn>
    </q-toolbar>
    <!-- <q-separator /> -->
    <q-card-section class="q-pa-none">
      <q-list separator id="scroll-items" class="scroll card-scroll__content">
        <template v-if="$q.platform.is.mobile">
          <tm-swipeitem v-for="(e,i) in rows" :key="i" leftValue="max" rightValue="111" v-touch-hold.mouse="()=>{onTouchHold(e)}">
            <template v-if="isRoutes.edit||isRoutes.trash" v-slot:right>
              <q-btn v-if="isRoutes.edit" no-caps class="q-btn--square" color="blue" @click="onEdit(e)">
                <q-icon name="edit" size="18px" />
              </q-btn>
              <q-btn v-if="isRoutes.trash" no-caps class="q-btn--square" color="negative" @click="onTrash(e)">
                <q-icon name="clear" size="18px" />
              </q-btn>
            </template>
            <q-item clickable v-ripple>
              <q-item-section>
                <q-item-label>{{e.name}}</q-item-label>
                <q-item-label caption lines="1">{{`${$t('global.code')}: ${e.code}`}}</q-item-label>
              </q-item-section>
              <q-item-section side>
                {{e.orders}}
              </q-item-section>
            </q-item>
          </tm-swipeitem>
        </template>
        <template v-else>
          <q-item clickable v-ripple v-for="(e,i) in rows" :key="i">
            <q-item-section>
              <q-item-label>{{e.name}}</q-item-label>
              <q-item-label caption lines="1">{{`${$t('global.code')}: ${e.code}`}}</q-item-label>
            </q-item-section>
            <q-item-section> {{e.orders}} </q-item-section>
            <q-item-section side>
              <q-icon name="edit" color="blue" size="18px" @click="onEdit(e)" />
            </q-item-section>
            <q-item-section side>
              <q-icon name="clear" color="negative" size="18px" @click="onTrash(e)" />
            </q-item-section>
          </q-item>
        </template>
      </q-list>
    </q-card-section>
  </q-card>
  <!-- Dialog Add -->
  <q-dialog v-model="isDialogAdd" :maximized="isMaximized">
    <add-item v-model:dialog="isDialogAdd" v-model:maximized="isMaximized" />
  </q-dialog>
  <!-- Dialog Actions -->
  <q-dialog v-model="isDialogTouchHold" position="bottom">
    <q-card class="full-width" style="border-radius:initial">
      <q-card-section class="q-pa-none">
        <q-list separator>
          <q-item clickable v-ripple class="text-center" @click="()=>{isDialogTouchHold=!isDialogTouchHold;onEdit()}">
            <q-item-section>{{$t('global.edit')}}</q-item-section>
          </q-item>
          <q-item clickable v-ripple class="text-center" @click="()=>{isDialogTouchHold=!isDialogTouchHold;onTrash()}">
            <q-item-section>
              <q-item-label v-if="pagination.flag" class="text-red">{{$t('global.trash')}}</q-item-label>
              <q-item-label v-else class="text-warning">{{$t('global.recover')}}</q-item-label>
            </q-item-section>
          </q-item>
        </q-list>
        <q-separator size="3px" />
        <q-list>
          <q-item clickable v-ripple class="text-center" v-close-popup>
            <q-item-section>
              <q-item-label>{{$t('global.cancel')}}</q-item-label>
            </q-item-section>
          </q-item>
        </q-list>
      </q-card-section>
    </q-card>
  </q-dialog>
</template>

<script>
import { defineComponent, defineAsyncComponent, ref, computed, onBeforeUnmount } from "vue";
import { useRouter } from 'vue-router'
import { useStore } from 'vuex'
import { useQuasar } from 'quasar'
import { useI18n } from 'vue-i18n'
import { normalize } from '../../utils/search'
export default defineComponent({
  name: "TypesIndex",
  components: {
    tmSwipeitem: defineAsyncComponent(() => import('components/tm-swipe-item/index.vue')),
    addItem: defineAsyncComponent(() => import('./add.vue')),
  },
  setup () {
    const $router = useRouter()
    const $store = useStore()
    const $q = useQuasar()
    const { t } = useI18n({ useScope: 'global' })
    const isDialogAdd = ref(false)
    const isMaximized = ref(true)
    const isDialogTouchHold = ref(false)
    const isFilter = ref(false)
    const selected = ref([])
    const isRoutes = ref({
      add: $router.hasRoute('manager-types-add'),
      edit: $router.hasRoute('manager-types-edit'),
      trash: $router.hasRoute('manager-types-trash')
    })
    const pagination = ref({
      filter: '',
      key: 'category',
      sortBy: 'orders',
      descending: false,
      page: 1,
      rowsPerPage: 10,
      // rowsNumber: 1,
      flag: 1
    })
    const onFetch = () => {
      selected.value = []
      if (pagination.value.filter) {
        return $store.state.types.items.filter(x =>
          x.key == pagination.value.key &&
          x.flag == pagination.value.flag &&
          (normalize(x.name)) === normalize(pagination.value.filter) || x.code === normalize(pagination.value.filter))
      } else {
        return $store.state.types.items.filter(x => x.key == pagination.value.key && x.flag == pagination.value.flag)
      }
    }

    // computed
    const rows = computed(() => onFetch())
    let Reset = null
    let timer
    function finalize (reset) {
      if (timer) clearTimeout(timer)
      timer = setTimeout(() => {
        reset()
      }, 3000)
    }
    onBeforeUnmount(() => {
      clearTimeout(timer)
    })
    return {
      isDialogAdd, isMaximized, isDialogTouchHold, rows, selected, isRoutes, isFilter, pagination,
      onSlideLeft ({ reset }) {
        // $q.notify('Left action triggered. Resetting in 1 second.')
        // finalize(reset)
        if (Reset) Reset()
        Reset = reset
        finalize(reset)
      },
      onSlideRight ({ reset }) {
        // $q.notify('Right action triggered. Resetting in 1 second.')
        // finalize(reset)
        if (Reset) Reset()
        Reset = reset
        finalize(reset)
      },
      onChangeFlag: (val) => {
        isFilter.value = false
        selected.value = []
        pagination.value.flag = val
        onFetch({ pagination: pagination.value }).then(x => data.value = x)
      },
      onAdd: () => {
        if (!isRoutes.value.add) return
        $store.dispatch('types/set')
        if ($q.platform.is.mobile) {
          isDialogAdd.value = true
          isMaximized.value = true
        } else if ($store.state.app.isDialog.add) {
          isDialogAdd.value = true
          isMaximized.value = false
        } else {
          $router.push('add')
        }
      },
      onEdit: (val) => {
        if (!isRoutes.value.edit) return
        if (val) selected.value = [val]
        $store.dispatch('types/set', selected.value[0]).then(x => selected.value = [])
        if ($q.platform.is.mobile) {
          isDialogAdd.value = true
          isMaximized.value = true
        } else if ($store.state.app.isDialog.add) {
          isDialogAdd.value = true
          isMaximized.value = false
        } else {
          $router.push('add')
        }
      },
      onTrash: (val) => {
        if (!isRoutes.value.trash) return
        $q.dialog({
          title: t('messageBox.warning'),
          message: pagination.value.flag ? t('messageBox.trash') : t('messageBox.recover'),
          cancel: true,
          ok: {
            label: t('global.accept'),
            flat: true,
            color: 'primary',
            noCaps: true
          },
          cancel: {
            label: t('global.cancel'),
            flat: true,
            color: 'blue-grey',
            noCaps: true
          }
        }).onOk(() => {
          if (val) selected.value = [val]
          $store.dispatch('types/patch', { _id: selected.value.map(x => x._id) }).then(x => { selected.value = [] })
        })
      },
      onTouchHold (val) {
        if (val) selected.value = [val]
        isDialogTouchHold.value = !isDialogTouchHold.value
      }
    }
  }
})
</script>
