<template>
  <q-card v-if="$q.platform.is.mobile" flat>
    <q-toolbar>
      <div v-if="$route.path!=='/manager/users/view'" class="col-auto">
        <q-btn flat dense icon="arrow_back" v-close-popup />
      </div>
      <q-toolbar-title>{{$t('route.users')}}</q-toolbar-title>
      <q-btn v-if="isRoutes.add" icon="add" flat round dense color="blue" @click="onAdd" />
      <q-btn v-if="isRoutes.add" flat round dense icon="file_upload" color="indigo" @click="onImport" />
      <q-btn icon="filter_list" flat round dense color="teal">
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
          <div class="row">
            <div class="col-12">
              <q-select v-model="group" input-debounce="200" :dense="$store.getters.dense.input" :options-dense="$store.getters.dense.input"
                        :options="groups" :label="$t('users.group')" @update:model-value="onChangeGroup">
                <template v-slot:selected-item="scope">
                  {{$t(`users.${scope.opt.code}`)}}
                </template>
                <template v-slot:option="scope">
                  <q-item v-bind="scope.itemProps">
                    <q-item-section>
                      <q-item-label>{{$t(`users.${scope.opt.code}`)}}</q-item-label>
                    </q-item-section>
                  </q-item>
                </template>
              </q-select>
            </div>
          </div>
          <div v-if="isRoutes.trash" class="row">
            <div class="col-auto self-center">{{$t("global.status")}}</div>
            <q-space />
            <div class="col-auto">
              <q-toggle v-model="pagination.enable" @update:model-value="onChangeEnable" />
            </div>
          </div>
        </q-menu>
      </q-btn>
    </q-toolbar>
    <!-- <q-separator /> -->
    <q-card-section class="q-pa-none">
      <q-list separator ref="refListTarget" id="scroll-items" class="scroll card-scroll__content">
        <q-infinite-scroll ref="refScrollTarget" @load="onScrollLoad" :offset="250" :scroll-target="refListTarget">
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
                <q-item-label>{{e.fullName}}</q-item-label>
                <q-item-label caption lines="1">
                  <span>{{$t('users.dateBirth')}}: </span>
                  <span class="text-green">{{e.dateBirth?$moment(e.dateBirth).format($store.getters.format.date):''}}</span>
                </q-item-label>
                <q-item-label caption lines="1">
                  <span>{{$t('users.gender')}}: </span>
                  <span class="text-green">{{$t(`gender.${e.gender}`)}}</span>
                </q-item-label>
                <q-item-label caption lines="1">
                  <span>{{$t('users.region')}}: </span>
                  <span class="text-green">{{onGetRegion(e.region)}}</span>
                </q-item-label>
                <q-item-label caption lines="1">
                  <span>{{$t('users.lastLogin')}}: </span>
                  <span v-if="e.lastLogin" class="text-green">
                    {{$moment(e.lastLogin).format(`${$store.getters.format.date} ${$store.getters.format.time}`)}}
                  </span>
                  <span v-else class="text-warning">
                    {{$t('table.noData')}}
                  </span>
                </q-item-label>
                <q-item-label caption lines="1">
                  <span>{{$t('users.lastChangePass')}}: </span>
                  <span v-if="e.lastChangePass" class="text-green">
                    {{$moment(e.lastChangePass).format(`${$store.getters.format.date} ${$store.getters.format.time}`)}}
                  </span>
                  <span v-else class="text-warning">
                    {{$t('table.noData')}}
                  </span>
                </q-item-label>
              </q-item-section>
              <!-- <q-item-section side top>
                <span>{{e.address}}</span>
              </q-item-section> -->
            </template>
            <q-separator v-if="i>0" />
            <q-item>
              <q-item-section avatar>
                <q-avatar rounded size="42px">
                  <q-img v-if="e.avatar&&e.avatar.length" :src="e.avatar[0].url">
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
                <q-item-label>{{e.username}}</q-item-label>
                <q-item-label caption lines="1">
                  <span>{{$t('roles.title')}}: </span>
                  <span v-for="role in e.userRoles" :key="role._id" :style="{color:role.color}" class="q-pr-xs">{{role.name}}</span>
                </q-item-label>
                <q-item-label caption lines="1">
                  <span>{{$t('users.group')}}: </span>
                  <span class="text-green">{{$t(`users.${e.group}`)}}</span>
                </q-item-label>
                <q-item-label caption lines="1">
                  <span>{{$t('users.email')}}: </span>
                  <span class="text-green">{{e.email}}</span>
                </q-item-label>
                <q-item-label caption lines="1">
                  <span>{{$t('users.phone')}}: </span>
                  <span class="text-green">{{e.phone}}</span>
                </q-item-label>
                <q-item-label caption lines="1">
                  <span>{{$t('users.personNumber')}}: </span>
                  <span class="text-green">{{e.personNumber}}</span>
                </q-item-label>
              </q-item-section>
              <q-item-section side>
                <q-icon name="verified" :color="e.verified?blue:''">
                  <q-popup-proxy>
                    <q-card>
                      <q-card-section>
                        {{e.verified?$t('users.verified'):$t('users.unverified')}}
                      </q-card-section>
                    </q-card>
                  </q-popup-proxy>
                </q-icon>
              </q-item-section>
            </q-item>
            <!-- <q-separator v-if="i<rows.length-1" /> -->
          </tm-swipeitem>
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
        <q-toolbar-title>{{$t('route.users')}}</q-toolbar-title>
        <q-btn v-if="isRoutes.add" flat round dense icon="add" color="blue" @click="onAdd">
          <q-tooltip>{{$t("global.add")}}</q-tooltip>
        </q-btn>
        <q-btn v-if="isRoutes.add" flat round dense icon="file_upload" color="indigo" @click="onImport">
          <q-tooltip>{{$t("files.openFile")}}</q-tooltip>
        </q-btn>
        <q-btn v-if="isRoutes.trash && selected.length>0&&pagination.enable" flat round dense color="negative" icon="delete" @click="onTrash()">
          <q-tooltip>{{$t("global.delete")}}</q-tooltip>
        </q-btn>
        <q-btn v-if="isRoutes.trash && selected.length>0&&!pagination.enable" flat round dense color="warning" icon="restore_page" @click="onTrash()">
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
                <q-select v-model="group" input-debounce="200" :dense="$store.getters.dense.input" :options-dense="$store.getters.dense.input"
                          :options="groups" :label="$t('users.group')" @update:model-value="onChangeGroup">
                  <template v-slot:selected-item="scope">
                    {{$t(`users.${scope.opt.code}`)}}
                  </template>
                  <template v-slot:option="scope">
                    <q-item v-bind="scope.itemProps">
                      <q-item-section>
                        <q-item-label>{{$t(`users.${scope.opt.code}`)}}</q-item-label>
                      </q-item-section>
                    </q-item>
                  </template>
                </q-select>
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
                <q-toggle v-model="pagination.enable" @update:model-value="onChangeEnable" />
              </div>
            </div>
          </q-menu>
        </q-btn>
      </q-toolbar>
    </template>
    <template v-slot:header="props">
      <q-tr :props="props">
        <q-th auto-width>
          <q-checkbox v-model="props.selected" indeterminate-value="some" :dense="$store.getters.dense.table" />
        </q-th>
        <q-th v-for="col in props.cols" :key="col.name" :props="props">
          <span class="text-bold">{{$t(col.label)}}</span>
        </q-th>
        <q-th v-if="isRoutes.edit||isRoutes.trash" auto-width>#</q-th>
      </q-tr>
    </template>
    <template v-slot:body="props">
      <q-tr :props="props">
        <q-td auto-width>
          <q-checkbox v-model="props.selected" color="primary" :dense="$store.getters.dense.table" />
        </q-td>
        <q-td key="username" :props="props">{{props.row.username}}</q-td>
        <q-td key="fullName" :props="props">{{props.row.fullName}}</q-td>
        <q-td key="email" :props="props">{{props.row.email}}</q-td>
        <q-td key="phone" :props="props">{{props.row.phone}}</q-td>
        <q-td key="roles" :props="props" class="q-gutter-xs">
          <!-- <q-badge :style="{backgroundColor:props.row.color}">{{ props.row.name }}</q-badge> -->
          <template v-if="props.row.roles && props.row.roles.length > 0">
            <q-badge class="bri" v-for="(e,i) in props.row.userRoles" :key="i" :style="{backgroundColor:e.color}">
              {{e.name}}
            </q-badge>
          </template>
          <q-badge class="bri" v-else color="blue-grey-10">{{$t("global.undefined")}}</q-badge>
          <!-- {{ props.row.roles.length>0?props.row.roles.length:$t('global.undefined') }} -->
        </q-td>
        <q-td key="verified" :props="props">
          <!-- {{ props.row.verified }} -->
          <!-- <q-avatar icon="done" :text-color="props.row.verified?'green':'blue-grey-10'" /> -->
          <q-avatar icon="verified" :text-color="props.row.verified?blue:''">
            <q-popup-proxy>
              <q-card>
                <q-card-section>
                  {{props.row.verified?$t('users.verified'):$t('users.unverified')}}
                </q-card-section>
              </q-card>
            </q-popup-proxy>
          </q-avatar>
        </q-td>
        <q-td v-if="isRoutes.edit || isRoutes.trash" auto-width class="text-center">
          <q-btn v-if="pagination.enable" flat round dense color="green" icon="vpn_key" :loading="loadingResetPassword"
                 :size="$store.getters.dense.table?'sm':'md'" @click="onResetPassword(props.row)">
            <q-tooltip>{{$t("users.resetPassword")}}</q-tooltip>
          </q-btn>
          <q-btn v-if="isRoutes.edit" flat round dense icon="edit" color="light-green" :size="$store.getters.dense.table?'sm':'md'"
                 @click="onEdit(props.row)">
            <q-tooltip>{{$t("global.update")}}</q-tooltip>
          </q-btn>
          <template v-if="isRoutes.trash">
            <q-btn v-if="pagination.enable" flat round dense color="negative" icon="clear" :size="$store.getters.dense.table?'sm':'md'"
                   @click="onTrash(props.row)">
              <q-tooltip>{{$t("global.lock")}}</q-tooltip>
            </q-btn>
            <q-btn v-else flat round dense color="amber" icon="restore" :size="$store.getters.dense.table?'sm':'md'" @click="onTrash(props.row)">
              <q-tooltip>{{$t("global.unlock")}}</q-tooltip>
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
  <!-- Dialog Import -->
  <q-dialog v-model="isDialogImport" :maximized="isMaximized" persistent>
    <import-item v-model:dialog="isDialogImport" v-model:maximized="isMaximized" />
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
              <q-item-label v-if="pagination.enable" class="text-red">{{$t('global.trash')}}</q-item-label>
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
import { regionConstant } from 'boot/i18n'
export default defineComponent({
  name: "UserIndex",
  components: {
    addItem: defineAsyncComponent(() => import('./add.vue')),
    importItem: defineAsyncComponent(() => import('./import.vue')),
    tmSwipeitem: defineAsyncComponent(() => import('components/tm-swipe-item/index.vue'))
  },
  setup () {
    const $router = useRouter()
    const $store = useStore()
    const $q = useQuasar()
    const { t } = useI18n({ useScope: 'global' })
    const isDialogAdd = ref(false)
    const isDialogImport = ref(false)
    const isDialogTouchHold = ref(false)
    const isMaximized = ref(false)
    const loadingResetPassword = ref(false)
    const refScrollTarget = ref(null)
    const refListTarget = ref(null)
    const isFilter = ref(false)
    const selected = ref([])
    const isRoutes = ref({
      add: $router.hasRoute('manager-users-add'),
      edit: $router.hasRoute('manager-users-edit'),
      trash: $router.hasRoute('manager-users-trash')
    })
    const pagination = ref({
      filter: '',
      group: 'client',
      sortBy: 'level',
      descending: false,
      page: 1,
      rowsPerPage: 10,
      rowsNumber: 1,
      enable: true
    })
    const visibleColumns = ref(['email', 'phone', 'roles', 'verified'])
    const columns = ref([
      { name: 'username', field: 'username', label: 'users.username', align: 'left', sortable: true, required: true },
      { name: 'fullName', field: 'fullName', label: 'users.fullName', align: 'left', sortable: true, required: true },
      { name: 'email', field: 'email', label: 'users.email', align: 'left', sortable: true },
      { name: 'phone', field: 'phone', label: 'users.phone', align: 'right', sortable: true },
      { name: 'roles', field: 'roles', label: 'roles.title', align: 'left', sortable: true },
      { name: 'verified', field: 'verified', label: 'users.verified', align: 'left', sortable: true }
    ])
    const groups = computed(() => $store.state.types.items.filter(x => x.key === 'userGroup'))
    const group = ref(groups.value ? groups.value[0] : { code: 'client', name: 'Client' }) // client or manager
    const data = ref([])
    const rows = computed(() => data.value)
    const onFetch = (opt) => {
      const { page, rowsPerPage, sortBy, descending, group } = opt.pagination
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
      pagination.value.descending = descending
      pagination.value.group = group
      return $store.dispatch('users/get', opt.pagination).then(x => {
        pagination.value.rowsNumber = x.rowsNumber
        pagination.value.totalPage = Math.ceil(pagination.value.rowsNumber / pagination.value.rowsPerPage)
        return x.data
      })
    }
    if ($store.state.auth.token) {
      onFetch({ pagination: pagination.value }).then(x => data.value = x)
    }
    return {
      isDialogAdd, isDialogImport, isDialogTouchHold, isMaximized, loadingResetPassword, refListTarget, refScrollTarget,
      isFilter, rows, selected, group, groups, pagination, isRoutes, onFetch, visibleColumns, columns,
      onFilter: (val) => {
        pagination.value.filter = val
        onFetch({ pagination: pagination.value }).then(x => { data.value = x })
      },
      onChangeEnable (val) {
        isFilter.value = false
        selected.value = []
        pagination.value.enable = val
        onFetch({ pagination: pagination.value }).then(x => { data.value = x })
      },
      onChangeGroup () {
        pagination.value.group = group.value.code
        onFetch({ pagination: pagination.value }).then(x => {
          data.value = x
          isFilter.value = false
        })
      },
      onColumns (val) {
        var i = visibleColumns.value.indexOf(val)
        if (i < 0) visibleColumns.value.push(val)
        else visibleColumns.value.splice(i, 1)
      },
      onGetRegion (region) {
        const rs = regionConstant.find(x => x.id === parseInt(region))
        if (rs) return rs.name_l
        else return region
      },
      onAdd: () => {
        if (!isRoutes.value.add) return
        $store.dispatch('users/set')
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
      onImport: () => {
        if (!isRoutes.value.add) return
        if ($q.platform.is.mobile) {
          isDialogImport.value = true
          isMaximized.value = true
        } else if ($store.state.app.isDialog.import) {
          isDialogImport.value = true
          isMaximized.value = false
        } else {
          $router.push('import')
        }
      },
      onEdit: (val) => {
        if (!isRoutes.value.edit) return
        if (val) selected.value = [val]
        $store.dispatch('users/set', selected.value[0]).then(x => selected.value = [])
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
          message: pagination.value.enable ? t('messageBox.lock') : t('messageBox.unlock'),
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
          $store.dispatch('users/patch', { _id: selected.value.map(x => x._id) }).then(x => { selected.value = [] })
        })
      },
      onResetPassword (val) {
        $q.dialog({
          title: t('messageBox.warning'),
          message: t('messageBox.resetPassword', { username: val.email }),
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
          loadingResetPassword.value = true
          $store.dispatch('users/resetPassword', {
            _id: val._id,
            password: $store.state.app.passwordResetDefault ? $store.state.app.passwordResetDefault : null
          }).then((x) => {
            $q.notify({ color: 'teal', message: t('users.msgResetPassword', { username: val.email, password: x.password }) })
            selected.value = []
            loadingResetPassword.value = false
          })
        }).onCancel(() => { selected.value = [] })
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
