<template>
  <q-card flat style="min-width:60%">
    <q-toolbar v-if="$q.platform.is.mobile">
      <div class="col-auto">
        <q-btn flat dense icon="arrow_back" v-close-popup />
      </div>
      <q-toolbar-title>{{$t('global.import')}} {{$t("route.news").toLowerCase()}}</q-toolbar-title>
      <q-btn icon="offline_pin" flat round dense color="blue" class="q-mr-sm" @click="onSubmit(1)" />
      <q-btn icon="draw" flat round dense color="amber" @click="onSubmit(0)" />
    </q-toolbar>
    <q-toolbar v-else>
      <q-toolbar-title>{{$t('global.import')}} {{$t("route.users").toLowerCase()}}</q-toolbar-title>
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
    <q-card-section class="q-pa-none">
    </q-card-section>
  </q-card>
</template>

<script>
import { defineComponent } from "vue";

export default defineComponent({
  name: "NewsImport",
  props: {
    dialog: { type: Boolean, default: false },
    maximized: { type: Boolean, default: false }
  },
  setup () {
    return {

    }
  }
})
</script>
