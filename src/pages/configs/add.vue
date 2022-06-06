<template>
  <q-card flat style="min-width:50%">
    <q-toolbar v-if="$q.platform.is.mobile">
      <div class="col-auto">
        <q-btn flat dense icon="arrow_back" v-close-popup />
      </div>
      <q-toolbar-title>
        {{data._id?`${$t('route.edit')} ${$t("route.configs").toLowerCase()}`:`${$t('route.add')} ${$t("route.configs").toLowerCase()}`}}
      </q-toolbar-title>
      <q-btn icon="offline_pin" flat round dense color="blue" class="q-mr-sm" @click="onSubmit(1)" />
      <q-btn icon="draw" flat round dense color="amber" @click="onSubmit(0)" />
    </q-toolbar>
    <q-toolbar v-else>
      <q-toolbar-title>
        {{data._id?`${$t('route.edit')} ${$t("route.configs").toLowerCase()}`:`${$t('route.add')} ${$t("route.configs").toLowerCase()}`}}
      </q-toolbar-title>
      <q-btn icon="offline_pin" flat round dense color="blue" class="q-mr-sm" @click="onSubmit(1)">
        <q-tooltip>{{data._id?$t('global.update'):$t('global.add')}}</q-tooltip>
      </q-btn>
      <q-btn icon="draw" flat round dense color="amber" @click="onSubmit(0)">
        <q-tooltip>{{$t('global.drafts')}}</q-tooltip>
      </q-btn>
      <q-btn v-if="dialog" flat round dense :color="$store.state.app.darkMode?'':'grey-7'"
             :icon="maximized?'fullscreen_exit':'fullscreen'" :disable="$store.state.app.loading.post||$store.state.app.loading.put"
             @click="$emit('update:maximized',!maximized)">
        <q-tooltip>
          {{maximized?$t('global.normalScreen'):$t('global.fullScreen')}}
        </q-tooltip>
      </q-btn>
      <q-btn v-if="dialog" flat round dense icon="close" :disable="$store.state.app.loading.post||$store.state.app.loading.put" v-close-popup>
        <q-tooltip>{{$t('global.cancel')}}</q-tooltip>
      </q-btn>
      <q-btn v-else flat round dense icon="reply" :disable="$store.state.app.loading.post||$store.state.app.loading.put"
             @click="$router.push('view')">
        <q-tooltip>{{$t('global.back')}}</q-tooltip>
      </q-btn>
    </q-toolbar>
    <q-separator />
    <q-card-section>
      <q-form ref="form">
        <!-- <q-scroll-area class="card-scroll__content-add"> -->
        <div class="row">
          <div class="col-12">
            <q-input v-model.trim="data.key" :dense="$store.getters.dense.input" :label="$t('global.key')" debounce="500" v-lowerCase
                     :rules="[v=>v&&v.length>0||$t('error.required'),v=>checkExistKey(v)]" />
          </div>
        </div>
        <div class="row">
          <div class="col-12">
            <q-input v-model.trim="data.value" :dense="$store.getters.dense.input" :label="$t('global.value')"
                     :rules="[v=>v&&v.length>0||$t('error.required')]" />
          </div>
        </div>
        <!-- </q-scroll-area> -->
      </q-form>
    </q-card-section>
  </q-card>
</template>

<script>
import { defineComponent, ref } from "vue";
import { useStore } from 'vuex'
import { useI18n } from 'vue-i18n'
export default defineComponent({
  name: "ConfigsAdd",
  props: {
    dialog: { type: Boolean, default: false },
    maximized: { type: Boolean, default: false }
  },
  setup () {
    const $store = useStore()
    const { t } = useI18n({ useScope: 'global' })
    const form = ref(null)
    const data = ref({})
    const onReset = () => {
      return new Promise((resolve, reject) => {
        if ($store.state.configs.item) data.value = { ...$store.state.configs.item }
        resolve()
      }).then(() => { if (form.value) form.value.resetValidation() })
    }

    onReset()

    return {
      form, data,
      checkExistKey: async (val) => {
        if (val) {
          if ($store.state.configs.item._id)
            if ($store.state.configs.item.key === data.value.key) return

          const rs = await $store.dispatch('configs/exist', { key: val })
          return rs ? t('error.exist') : true
        }
      },
      onSubmit: () => {
        form.value.validate().then(valid => {
          if (valid) {
            if ($store.state.configs.item._id) $store.dispatch('configs/put', data.value)
            else $store.dispatch('configs/post', data.value).then(onReset())
          }
        })
      }
    }
  }
})
</script>
