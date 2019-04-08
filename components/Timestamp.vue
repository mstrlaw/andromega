<template>
  <time class="text-xs text-grey-dark">
    {{ localValue }}
  </time>
</template>

<script>
  import moment from 'moment'

  export default {
    name: 'Timestamp',
    props: {
      timestamp: {
        type: Number,
        default: 0
      }
    },
    data() {
      return {
        updateInterval: null,
        localValue: ''
      }
    },
    mounted() {
      this.localValue = moment().to(this.timestamp)

      this.updateInterval = setInterval( () => {
        this.localValue = moment().to(this.timestamp)
      }, 1000)
    },
    destroyed() {
      clearInterval(this.updateInterval)
    }
  }
</script>
