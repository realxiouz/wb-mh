<template>
  <div>
    <div class="header">
      <div :style="{ height: `${navBarHeight}px`, padding: navBarPadding }" >

			</div>
      <div style="height:300rpx;" class=""></div>
    </div>
    <shopro-login-modal />
    <shopro-tabbar />
  </div>
</template>

<script>
import { mapState } from 'vuex'
export default {
  computed: {
    ...mapState('device', ['navBarHeight', 'navBarPadding']),
  }
}
</script>

<style lang="scss">
.header{
  background: linear-gradient(180deg, rgba(239, 196, 128, 1) 0%, rgba(248, 220, 165, 1) 25%, rgba(255, 255, 255, 1) 98%);
}
</style>