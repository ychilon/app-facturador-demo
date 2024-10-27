<template>
  <f7-block :class="[configuration.header_waves === 1 ? 'display-flex' : 'display-none', theme.class_bg_body]" class="no-margin-top no-margin-vertical no-padding-horizontal">
    <div class="waves">
      <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 200 100">
        <path d="M0 0 A100 100 0 0 0 200 0 L200 0 L0 0 Z" fill="#2B614D"/>
      </svg>
    </div>
  </f7-block>
</template>

<script>
  import { general_functions } from "mixins_/general_functions"

  export default {
    name: "HeaderWaves",
    mixins: [ general_functions ],
    props: [ "title" ],
    data: function () {
      return {
        configuration: {},
        theme: {}
      }
    },
    mounted() {
      this.loadConfiguration()
    },
    async created() {
        await this.checkTheme()
    },
    methods: {
      loadConfiguration(){
        this.configuration = this.getInitialConfiguration()
      },
      checkTheme() {
          this.theme = this.getThemeSettings()
      }
    }
  }
</script>