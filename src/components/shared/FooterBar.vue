<template>
  <footer
    v-show="isFooterBarVisible"
    class="footer"
  >
    <div class="container-fluid">
      <div class="footer-copyright has-text-centered">
        <b>&copy; {{ year }}</b> Lennox Kamau
      </div>
    </div>
  </footer>
</template>

<script>
import { defineComponent } from '@vue/composition-api'
import { mapState } from 'vuex'

export default defineComponent({
  name: 'FooterBar',
  data () {
    return {
      year: new Date().getFullYear()
    }
  },
  computed: {
    ...mapState({
      isFooterBarVisible: state => state.system.isFooterBarVisible
    })
  }
})
</script>
