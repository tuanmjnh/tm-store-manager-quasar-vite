<template>
  <q-card v-if="$q.platform.is.mobile" flat>
    <q-toolbar>
      <div v-if="$route.path!=='/product/list/view'" class="col-auto">
        <q-btn flat dense icon="arrow_back" v-close-popup />
      </div>
      <q-toolbar-title>{{$t('route.product')}}</q-toolbar-title>
      <q-btn v-if="isRoutes.add" icon="add" flat round dense color="blue" @click="onAdd" />
      <q-btn icon="filter_list" flat round dense color="teal">
        <q-menu v-model="isFilter" class="q-pa-md">
          <!-- <div class="q-pa-md"> -->
          <div class="row">
            <div class="col-12">
              <q-input v-model="pagination.filter" :dense="$store.getters.dense.input" debounce="500" :placeholder="$t('global.search')"
                       @update:model-value="onFilter">
                <template v-slot:append>
                  <q-icon v-if="pagination.filter===''" name="search" />
                  <q-icon v-else name="clear" class="cursor-pointer" @click="onFilter('')" />
                </template>
              </q-input>
            </div>
          </div>
          <div class="row">
            <div class="col-12">
              <select-category v-model="pagination.categories" :categories="categories" option-value="_id" option-label="label" data-all maximized
                               :dense="$store.getters.dense.input" :labelTitle="$t('category.titleproduct')" :labelSelect="$t('category.select')"
                               :labelAll="$t('category.selectAll')" :labelClose="$t('global.cancel')" @on-selected="onSelectCategory"
                               @onCancel="isFilter=false" />
            </div>
          </div>
          <div v-if="isRoutes.trash" class="row">
            <div class="col-auto self-center">{{$t("global.status")}}</div>
            <q-space />
            <div class="col-auto">
              <q-toggle v-model="pagination.flag" :false-value="0" :true-value="1" @update:model-value="onChangeFlag" />
            </div>
          </div>
        </q-menu>
      </q-btn>
    </q-toolbar>
    <!-- <q-separator /> -->
    <q-card-section class="q-pa-none">
      <q-list separator id="scroll-items" class="scroll card-scroll__content">
        <q-infinite-scroll ref="refScrollTarget" @load="onScrollLoad" :offset="250">
          <tm-swipeitem v-for="(e,i) in rows" :key="i" leftValue="max" rightValue="111" v-touch-hold.mouse="()=>{onTouchHold(e)}">
            <template v-if="isRoutes.edit||isRoutes.trash" v-slot:right>
              <q-btn v-if="isRoutes.edit" no-caps class="q-btn--square" @click="onEdit(e)">
                <q-icon name="edit" color="blue" size="18px" />
              </q-btn>
              <q-btn v-if="isRoutes.trash" no-caps class="q-btn--square" @click="onTrash(e)">
                <q-icon name="clear" color="negative" size="18px" />
              </q-btn>
            </template>
            <template v-slot:left>
              <q-item-section class="q-pl-lg q-pr-lg">
                <q-item-label>{{e.title}}</q-item-label>
                <q-item-label caption lines="1">
                  <span>{{$t('global.code')}}: </span>
                  <span class="text-blue">{{e.code}}</span>
                </q-item-label>
                <q-item-label caption lines="1">
                  <span>{{$t('global.quantity')}}: </span>
                  <span class="text-green">{{parseInt(e.quantity).NumberFormat($store.getters.language)}}</span>
                </q-item-label>
                <q-item-label caption lines="1">
                  <span>{{$t('product.priceSale')}}: </span>
                  {{parseInt(e.price).NumberFormat($store.getters.language)}}
                  <q-badge color="red" transparent>{{$store.getters.unitPrice}}</q-badge>
                </q-item-label>
                <q-item-label caption lines="1">
                  <span>{{$t('product.priceImport')}}: </span>
                  <span class="text-purple">
                    {{parseInt(e.priceImport).NumberFormat($store.getters.language)}}
                    <q-badge color="red" transparent>{{$store.getters.unitPrice}}</q-badge>
                  </span>
                </q-item-label>
              </q-item-section>
            </template>
            <q-separator v-if="i>0" />
            <q-item>
              <q-item-section avatar>
                <q-avatar rounded size="42px">
                  <q-img v-if="e.images&&e.images.length&&e.images[0].length" :src="e.images[0]">
                    <template v-slot:error>
                      <div class="image-error absolute-full flex flex-center">
                        <q-icon name="insert_photo" />
                      </div>
                    </template>
                  </q-img>
                  <q-img v-else>
                    <div class="image-error absolute-full flex flex-center">
                      <q-icon name="insert_photo" />
                    </div>
                  </q-img>
                </q-avatar>
              </q-item-section>
              <q-item-section>
                <q-item-label>{{e.title}}</q-item-label>
                <q-item-label caption lines="1">
                  <span>{{$t('category.title')}}: </span>
                  <span v-for="cate in e.categoryList" :key="cate._id" :style="{color:cate.color}" class="q-pr-xs">{{cate.title}}</span>
                </q-item-label>
                <q-item-label caption lines="1">
                  <span>{{$t('global.code')}}: </span>
                  <span class="text-blue">{{e.code}}</span>
                </q-item-label>
                <q-item-label caption lines="1">
                  <span>{{$t('global.quantity')}}: </span>
                  <span class="text-green">{{parseInt(e.quantity).NumberFormat($store.getters.language)}}</span>
                </q-item-label>
                <q-item-label caption lines="1">
                  <span>{{$t('product.priceSale')}}: </span>
                  <span class="text-red">
                    {{parseInt(e.price).NumberFormat($store.getters.language)}}
                    <q-badge color="red" transparent>{{$store.getters.unitPrice}}</q-badge>
                  </span>
                </q-item-label>
              </q-item-section>
              <!-- <q-item-section side>
                {{e.order}}
              </q-item-section> -->
              <q-item-section side>
                {{e.unitType.name}}
              </q-item-section>
            </q-item>
          </tm-swipeitem>
          <!-- <tm-swipeitem v-for="e in rows" :key="e._id" :translateXValue="111">
            <template v-slot:right>
              <q-btn no-caps class="q-btn--square" @click="onEdit(e)">
                <q-icon name="edit" color="blue" size="18px" />
              </q-btn>
              <q-btn no-caps class="q-btn--square" @click="onTrash(e)">
                <q-icon name="clear" color="negative" size="18px" />
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
          </tm-swipeitem> -->
          <template v-slot:loading>
            <div class="row justify-center q-my-md">
              <q-spinner-dots color="primary" size="40px" />
            </div>
          </template>
        </q-infinite-scroll>
      </q-list>
    </q-card-section>
  </q-card>
  <q-table v-else :rows="rows" :columns="columns" row-key="_id" flat :visible-columns="visibleColumns" :dense="$store.getters.dense.table"
           selection="multiple" :no-data-label="$t('table.noData')" :filter="pagination.filter" binary-state-sort
           :loading="$store.state.app.loading.get||$store.state.app.loading.patch" v-model:selected="selected"
           :rows-per-page-label="$t('table.rowPerPage')" :selected-rows-label="()=>`${selected.length} ${$t('table.rowSelected')}`"
           :rows-per-page-options="$store.state.app.rowsPerPageOptions" v-model:pagination="pagination" @request="onFetch">
    <template v-slot:top="props">
      <q-toolbar class="q-pa-none">
        <q-toolbar-title>{{$t('route.product')}}</q-toolbar-title>
        <q-btn v-if="isRoutes.add" flat round dense icon="add" color="blue" @click="onAdd">
          <q-tooltip>{{$t("global.add")}}</q-tooltip>
        </q-btn>
        <q-btn v-if="isRoutes.trash && selected.length>0&&pagination.flag" flat round dense color="negative" icon="delete" @click="onTrash()">
          <q-tooltip>{{$t("global.delete")}}</q-tooltip>
        </q-btn>
        <q-btn v-if="isRoutes.trash && selected.length>0&&!pagination.flag" flat round dense color="warning" icon="restore_page" @click="onTrash()">
          <q-tooltip>{{$t("global.recover")}}</q-tooltip>
        </q-btn>
        <q-btn flat round dense :icon="props.inFullscreen?'fullscreen_exit':'fullscreen'" @click="props.toggleFullscreen">
          <q-tooltip>{{props.inFullscreen?$t("global.normalScreen"):$t("global.fullScreen")}}</q-tooltip>
        </q-btn>
        <q-btn icon="filter_list" flat round dense>
          <q-tooltip>{{$t('global.filter')}}</q-tooltip>
          <q-menu v-model="isFilter" class="q-pa-md">
            <div class="row">
              <div class="col-12">
                <q-input v-model="pagination.filter" :dense="$store.getters.dense.input" debounce="500" :placeholder="$t('global.search')"
                         @update:model-value="onFilter">
                  <template v-slot:append>
                    <q-icon v-if="pagination.filter===''" name="search" />
                    <q-icon v-else name="clear" class="cursor-pointer" @click="onFilter('')" />
                  </template>
                </q-input>
              </div>
            </div>
            <div class="row q-mb-sm">
              <div class="col-12">
                <select-category v-model="pagination.categories" :categories="categories" option-value="_id" option-label="label" data-all
                                 :dense="$store.getters.dense.input" :labelTitle="$t('category.titleproduct')" :labelSelect="$t('category.select')"
                                 :labelAll="$t('category.selectAll')" :labelClose="$t('global.cancel')" @on-selected="onSelectCategory" />
              </div>
            </div>
            <div class="row">
              <div class="col-auto self-center">{{$t("table.displayColumns")}}</div>
              <q-space />
              <div class="col-auto">
                <q-btn flat round dense icon="menu_open">
                  <q-menu fit>
                    <q-list dense style="min-width:200px">
                      <template v-for="(e,i) in columns">
                        <q-item clickable :key="i" v-if="!e.required" :active="visibleColumns.indexOf(e.name)>-1||false" @click="onColumns(e.name)">
                          <q-item-section>{{$t(e.label)}}</q-item-section>
                        </q-item>
                      </template>
                    </q-list>
                  </q-menu>
                </q-btn>
              </div>
            </div>
            <div v-if="isRoutes.trash" class="row">
              <div class="col-auto self-center">{{$t("global.status")}}</div>
              <q-space />
              <div class="col-auto">
                <q-toggle v-model="pagination.flag" :false-value="0" :true-value="1" @update:model-value="onChangeFlag" />
              </div>
            </div>
          </q-menu>
        </q-btn>
      </q-toolbar>
    </template>
    <template v-slot:header="props">
      <q-tr :props="props">
        <q-th auto-width>
          <q-checkbox v-if="props.multipleSelect" v-model="props.selected" indeterminate-value="some" :dense="$store.getters.dense.table" />
        </q-th>
        <q-th v-for="col in props.cols" :key="col.name" :props="props">
          <span v-if="$store.state.app.darkMode" class="text-bold">{{$t(col.label)}}</span>
          <span v-else class="text-bold text-blue-grey-10">{{$t(col.label)}}</span>
        </q-th>
        <q-th v-if="isRoutes.edit||isRoutes.trash" auto-width>#</q-th>
      </q-tr>
    </template>
    <template v-slot:body="props">
      <q-tr :props="props">
        <q-td auto-width>
          <q-checkbox v-model="props.selected" color="primary" :dense="$store.getters.dense.table" />
        </q-td>
        <q-td key="title" :props="props">
          {{props.row.title}}
        </q-td>
        <q-td key="code" :props="props">
          {{props.row.code}}
        </q-td>
        <q-td key="quantity" :props="props">
          <div class="q-pr-xs flex inline">{{parseInt(props.row.quantity).NumberFormat($store.getters.language)}}</div>
          <q-badge v-html="props.row.unitName" color="orange" transparent />
        </q-td>
        <q-td key="price" :props="props">
          <div class="q-pr-xs flex inline">{{parseInt(props.row.price).NumberFormat($store.getters.language)}}</div>
          <q-badge v-html="props.row.priceUnitName" color="blue" transparent />
        </q-td>
        <q-td key="priceDiscount" :props="props">
          <div class="q-pr-xs flex inline">{{parseInt(props.row.priceDiscount).NumberFormat($store.getters.language)}}</div>
          <q-badge v-html="props.row.priceUnitName" color="red" transparent />
        </q-td>
        <q-td key="priceImport" :props="props">
          <div class="q-pr-xs flex inline">{{parseInt(props.row.priceImport).NumberFormat($store.getters.language)}}</div>
          <q-badge v-html="props.row.priceUnitName" color="teal" transparent />
        </q-td>
        <q-td key="priceExport" :props="props">
          <div class="q-pr-xs flex inline">{{parseInt(props.row.priceExport).NumberFormat($store.getters.language)}}</div>
          <q-badge v-html="props.row.priceUnitName" color="indigo" transparent />
        </q-td>
        <q-td key="order" :props="props">
          {{props.row.order}}
        </q-td>
        <!-- <q-td key="roles" :props="props" class="q-gutter-xs">
            <template v-if="props.row.roles&&props.row.roles.length>0">
              <q-badge v-for="(item,index) in onFilterRoles(props.row.roles)" :key="index"
                :style="{backgroundColor:item.color}">
                {{item.label}}
              </q-badge>
            </template>
            <q-badge v-else color="blue-grey-10">{{$t('global.undefined')}}</q-badge>
          </q-td> -->
        <q-td v-if="isRoutes.edit||isRoutes.trash" auto-width class="text-center">
          <q-btn v-if="isRoutes.edit" flat round dense icon="edit" color="light-green" :size="$store.getters.dense.table?'sm':'md'"
                 @click="onEdit(props.row)">
            <q-tooltip>{{$t('global.update')}}</q-tooltip>
          </q-btn>
          <template v-if="isRoutes.trash">
            <q-btn v-if="pagination.flag" flat round dense color="negative" icon="clear" :size="$store.getters.dense.table?'sm':'md'"
                   @click="onTrash(props.row)">
              <q-tooltip>{{$t('global.lock')}}</q-tooltip>
            </q-btn>
            <q-btn v-else flat round dense color="amber" icon="restore" :size="$store.getters.dense.table?'sm':'md'" @click="onTrash(props.row)">
              <q-tooltip>{{$t('global.unlock')}}</q-tooltip>
            </q-btn>
          </template>
        </q-td>
      </q-tr>
    </template>
  </q-table>
  <!-- Dialog Add -->
  <q-dialog v-model="isDialogAdd" :maximized="isMaximized" persistent>
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
import { defineComponent, defineAsyncComponent, ref, computed } from "vue";
import { useRouter } from 'vue-router'
import { useStore } from 'vuex'
import { useQuasar } from 'quasar'
import { useI18n } from 'vue-i18n'
export default defineComponent({
  name: "ProductIndex",
  components: {
    addItem: defineAsyncComponent(() => import('./add.vue')),
    tmSwipeitem: defineAsyncComponent(() => import('components/tm-swipe-item/index.vue')),
    selectCategory: defineAsyncComponent(() => import('pages/category/components/select-category.vue'))
  },
  setup () {
    const $router = useRouter()
    const $store = useStore()
    const $q = useQuasar()
    const { t } = useI18n({ useScope: 'global' })
    const isDialogAdd = ref(false)
    const isMaximized = ref(true)
    const isDialogTouchHold = ref(false)
    const refScrollTarget = ref(null)
    const refListTarget = ref(null)
    const isFilter = ref(false)
    const selected = ref([])
    const categories = computed(() => $store.state.categories.all.product || [])
    // const pins = computed(() => $store.state.types.items.filter(x => x.key === 'pin_product'))
    // const units = computed(() => $store.state.types.items.filter(x => x.key === 'unit'))
    // const unitsPrice = computed(() => $store.state.types.items.filter(x => x.key === 'unit_price'))
    const isRoutes = ref({
      add: $router.hasRoute('product-list-add'),
      edit: $router.hasRoute('product-list-edit'),
      trash: $router.hasRoute('product-list-trash')
    })
    const pagination = ref({
      filter: '',
      page: 1,
      rowsPerPage: 15,
      rowsNumber: 1,
      totalPage: 1,
      sortBy: 'order',
      descending: false,
      categories: null,
      flag: 1
    })
    const visibleColumns = ref(['price', 'priceDiscount'])
    const columns = ref([
      { name: 'title', field: 'title', label: 'product.name', align: 'left', sortable: true, required: true }, // row => $t(`roles.${row.name}`)
      { name: 'code', field: 'code', label: 'product.code', align: 'left', sortable: true, required: true },
      { name: 'quantity', field: 'quantity', label: 'global.quantity', align: 'right', sortable: true, required: true },
      { name: 'price', field: 'price', label: 'product.priceSale', align: 'right', sortable: true },
      { name: 'priceDiscount', field: 'priceDiscount', label: 'product.priceDiscount', align: 'right', sortable: true },
      { name: 'priceImport', field: 'priceImport', label: 'product.priceImport', align: 'right', sortable: true },
      { name: 'priceExport', field: 'priceExport', label: 'product.priceExport', align: 'right', sortable: true },
      { name: 'order', field: 'order', label: 'global.order', align: 'right', sortable: true }
    ])
    const data = ref([])
    const rows = computed(() => data.value)
    const onFetch = (opt) => {
      const { page, rowsPerPage, sortBy, descending, categories } = opt.pagination
      if (opt.scroll) pagination.value.page = page
      else {
        if (refScrollTarget.value) {
          document.getElementById('scroll-items').scroll(0, 0)
          refScrollTarget.value.reset()
        }
        pagination.value.page = 1
      }
      pagination.value.rowsPerPage = rowsPerPage
      pagination.value.sortBy = sortBy
      pagination.value.categories = categories
      pagination.value.descending = descending
      return $store.dispatch('products/get', opt.pagination).then(x => {
        pagination.value.rowsNumber = x.rowsNumber
        pagination.value.totalPage = Math.ceil(pagination.value.rowsNumber / pagination.value.rowsPerPage)
        // data.value = data.value.concat(x.data)
        return x.data
      })
    }
    if ($store.state.auth.token) {
      onFetch({ pagination: pagination.value }).then(x => data.value = x)
      if (!$store.state.categories.all.product || $store.state.categories.all.product.length < 1)
        $store.dispatch('categories/get', { type: 'product', flag: 1, all: true })//.then((x) => { categories.value = x })
    }
    return {
      isDialogAdd, isMaximized, isDialogTouchHold, refListTarget, refScrollTarget, isFilter, rows, selected,
      categories, pagination, isRoutes, onFetch, visibleColumns, columns,
      onSelectCategory: (val) => {
        if (val) onFetch({ pagination: pagination.value }).then(x => {
          data.value = x
          isFilter.value = false
        })
      },
      onFilter: (val) => {
        pagination.value.filter = val
        onFetch({ pagination: pagination.value }).then(x => { data.value = x })
      },
      onChangeFlag: (val) => {
        isFilter.value = false
        selected.value = []
        pagination.value.flag = val
        onFetch({ pagination: pagination.value }).then(x => data.value = x)
      },
      onColumns: (val) => {
        var i = visibleColumns.value.indexOf(val)
        if (i < 0) visibleColumns.value.push(val)
        else visibleColumns.value.splice(i, 1)
      },
      onAdd: () => {
        if (!isRoutes.value.add) return
        $store.dispatch('products/set')
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
        $store.dispatch('products/set', selected.value[0]).then(x => selected.value = [])
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
      onTrash (val) {
        if (!isRoutes.value.trash) return
        $q.dialog({
          title: t('messageBox.warning'),
          message: pagination.value.flag ? t('messageBox.lock') : t('messageBox.unlock'),
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
          $store.dispatch('products/patch', { _id: selected.value.map(x => x._id) }).then(x => { selected.value = [] })
        })
      },
      onScrollLoad (index, done) {
        pagination.value.page = index + 1
        if (pagination.value.page <= pagination.value.totalPage)
          onFetch({ pagination: pagination.value, scroll: true }).then(x => {
            data.value = data.value.concat(x)
            done()
          })
        else {
          done()
        }
      },
      onTouchHold (val) {
        if (val) selected.value = [val]
        isDialogTouchHold.value = !isDialogTouchHold.value
      }
    }
  }
})
</script>
