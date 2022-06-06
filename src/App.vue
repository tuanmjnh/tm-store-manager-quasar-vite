<template>
  <loading-layout v-if="$store.state.auth.loading||$store.state.app.loading.get" />
  <component :is="component">
    <router-view />
  </component>
</template>
<script>
import { defineComponent, defineAsyncComponent, computed } from "vue";
import { useQuasar } from 'quasar'
import { useStore } from 'vuex'
export default defineComponent({
  name: "App",
  components: {
    loadingLayout: defineAsyncComponent(() => import('./layouts/loading/index.vue')),
    // light: defineAsyncComponent(() => import('layouts/light')),
    // login: defineAsyncComponent(() => import('layouts/login'))
  },
  setup () {
    const $q = useQuasar()
    const $store = useStore()
    const layout = computed(() => $store.getters.layout)
    const component = computed(() => {
      if ($store.state.auth.user) {
        return defineAsyncComponent(() => import(`./layouts/${layout.value}/index.vue`))
      } else {
        return defineAsyncComponent(() => import('./layouts/fake-layout.vue'))
      }
    })
    $store.commit('settings/INIT')
    $q.dark.set($store.getters.darkMode)

    // $store.dispatch('configs/post', { key: 'logger', value: 1 }).then(x => {
    //   console.log(x)
    // })
    // $store.dispatch('configs/get').then(x => {
    //   console.log(x)
    // })
    return { component }
  }
});
</script>
