<template>
  <div class="loading-layout">
    <!-- calc(100vw - 300px) -->
    <div class="loading-content">
      <q-spinner-puff color="blue" size="20em" :thickness="1" />
    </div>
  </div>
</template>

<script>
import { defineComponent } from 'vue';
export default defineComponent({
  name: 'LoadingLayout',
  setup () {
    // const width = computed(() => {
    //   const pageContainerEL = document.getElementById("page-container")
    //   return pageContainerEL.offsetWidth
    // })
    // const width = window.screen.width
    // const height = window.screen.height
    // const style = {
    //   height: `calc(100vh - 50px)`,
    //   display: 'flex',
    //   justifyContent: 'center',
    //   alignItems: 'center'
    // }
    // return { width }
  }
})
</script>

<style>
/* .mobile .loading-layout {
  position: absolute;
  width: 100%;
  left: 0;
}
.loading-layout {
  position: absolute;
  left: 50%;
} */
.loading-layout {
  position: absolute;
  z-index: 6001;
  width: 100%;
}
.loading-content {
  height: calc(100vh - 50px);
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>
