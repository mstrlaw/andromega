<template>
  <div class="player-wrapper">
    <div ref="player" />
  </div>
</template>

<script>
  const YTPlayer = require('yt-player')

  export default {
    name: 'Player',
    data() {
      return {
        player: null,
        muted: true,
        playing: true,
        src: 'kovd7OzcU9s'
      }
    },
    mounted() {
      setTimeout( () => {
        this.initPlayer()

      }, 1000)
    },
    methods: {
      initPlayer() {
        this.player = new YTPlayer(this.$refs.player, {
          controls: false,
          autoplay: true,
          related: false,
          annotations: false
        })

        // this.player.mute()

        // this.player.seek(0)

        this.player.load(this.src)

        this.player.play()

        this.player.on('playing', () => {
          // this.$toast.success('OK')
        })
      },
    }
  }
</script>

<style scoped>
  .player-wrapper{
    position: relative;
    height: 1px;
    width: 1px;
    overflow: hidden;
    opacity: 0;
    z-index: -1;
  }
</style>
