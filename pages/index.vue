<template>
  <section class="app-container">
    
    <audio id="audio" controls>
        <source
          id="audioSource"
          type="audio/mpeg"
        >
      </audio>

    <!--div class="flex fixed justify-start pin-t p-2 w-full z-10">
      <a
        href="#"
        class="bg-blue hover:bg-blue-dark text-white font-bold text-center no-underline py-2 px-4 mx-2 w-16 rounded"
        @click.prevent="togglePanel('chat')"
      >Chat</a>
    </div-->

    <div
      :class="activePanel === 'chat' ? 'visible': ''"
      class="panel chat-pannel"
    >
      <div
        v-for="(message, index) in messagesList"
        :key="index"
        :class="index % 2 > 0 ? 'bg-grey-lightest' : ''"
        class="flex flex-col my-0 p-2 pb-2"
      >
        <span class="text-lg font-medium">{{ message.text }}</span>
        <Timestamp :timestamp="message.timestamp" />
      </div>
    </div>

    <div
      :class="activePanel === 'chat' ? 'visible': ''"
      class="chat-input flex fixed justify-between pin-b p-2 w-full bg-grey-lightest z-10"
    >
      <input
        ref="messageInput"
        type="text"
        class="p-2 border rounded w-4/5"
        placeholder="Type in a message or provide microphone access and speak"
        v-model="newMessage"
        @keyup.enter="submitMessage"
      >
      <a
        href="#"
        class="btn bg-blue hover:bg-blue-dark text-white font-bold text-center no-underline py-2 px-4 mx-2 w-full rounded"
        @click.prevent="submitMessage"
      >Send</a>
      <a
        href="#"
        class="btn bg-grey hover:bg-grey-dark text-white font-bold text-center no-underline py-2 px-4 mx-2 w-full rounded"
        @click.prevent="toggleRecognition"
      >
        <img
          v-if="listening"
          src="https://image.flaticon.com/icons/svg/149/149427.svg" alt="Active Mic"
        >
        <img
          v-else
          src="https://image.flaticon.com/icons/svg/149/149428.svg" alt="Innactive Mic"
        >
      </a>
    </div>
  </section>
</template>

<script>
  import io from 'socket.io-client'
  import moment from 'moment'
  
  import Timestamp from '@/components/Timestamp'
  import Player from '@/components/Player'

  export default {
    name: 'Home',
    components: {
      Timestamp,
      Player
    },
    data() {
      return {
        newMessage: '',
        socket: null,
        messagesList: [],
        sceneElements: '',
        activePanel: 'chat',
        listening: false,
        speaking: false,
        recognition: null,
        audioEl: null,
        sourceEl: null,
        ttsString: 'Hello+you+weird+duck',
        voices: [
          'GB_KateVoice',
        ]

      }
    },
    // created() {
    //   this.socket = io(process.env.SOCKET_ENDPOINT)

    //   this.socket
    //     .on('connect', () => {
    //       console.log('connected to server')
    //     })
    //     .on('UPDATE_MESSAGES', (msg) => {
    //       this.updateMessages(msg)
    //     })
    // },
    mounted() {

      this.loadMessages()

      window.SpeechRecognition = window.webkitSpeechRecognition || window.SpeechRecognition
      
      if (window.SpeechRecognition) {
        this.initiateSpeechRecognition()
      }

      this.audioEl = document.getElementById('audio')

      this.sourceEl = document.getElementById('audioSource')

      this.audioEl.onended = () => {
        this.startRecognition()
      }
    },
    methods: {
      initiateSpeechRecognition() {
        this.recognition = new window.SpeechRecognition()

        this.recognition.continuous = true

        this.recognition.onresult = (event) => {
          let speechToText = event.results[event.results.length - 1][0].transcript
          console.log('finished StT processing')

          if (speechToText === 'clear messages' || speechToText === 'reset messages' || speechToText === 'reset room') {
            this.messagesList = [{
              text: 'Hello, type in something or speak to your microphone (Chrome only)',
              timestamp: moment().valueOf()
            }]
            this.clearMessages()
          } else {
            this.updateMessages({
              text: speechToText,
              timestamp: moment().valueOf()
            })

            this.saveMessages()
          }

          
          // this.socket.emit('NEW_MESSAGE', {
          //   text: speechToText,
          //   timestamp: moment().valueOf()
          // })

        }
        
        this.startRecognition()

      },
      togglePanel(panel) {
        this.activePanel = panel
      },
      /**
      * Chat related methods
      */
      submitMessage() {
        if (this.newMessage.trim().length > 0) {
          this.socket.emit('NEW_MESSAGE', {
            text: this.newMessage,
            timestamp: moment().valueOf()
          })
          this.newMessage = ''
          this.$refs.messageInput.focus()
        }
      },
      updateMessages(msg) {
        this.messagesList.push(msg)

        this.loadTTSCall(msg.text)
        
        setTimeout( () => {
          window.scrollTo(0, document.body.scrollHeight + 120)
        }, 150)
      },
      loadTTSCall(msg) {
        let tmp = msg.split(' ').join('+')

        let endpoint = `https://text-to-speech-demo.ng.bluemix.net/api/v1/synthesize?text=%3Cvoice-transformation+type%3D%22Custom%22+pitch_range%3D%2280%25%22+glottal_tension%3D%22-80%25%22%3E+${ tmp }+%3C%2Fvoice-transformation%3E&voice=en-US_AllisonVoice&download=true&accept=audio%2Fmp3`

        this.stopRecognition()

        this.sourceEl.src = endpoint

        this.audioEl.load() //Preload the audio from Watson

        this.loadAudio()
          .then( () => {
            this.audioEl.play()
          })
      },
      loadAudio() {
        return new Promise((resolve, reject) => {
          this.audioEl.addEventListener('canplay', resolve)
        })
      },
      toggleRecognition() {
        if (this.listening) {
          this.stopRecognition()
        } else {
          this.startRecognition()
        }
      },
      startRecognition() {
        this.recognition.start()
        this.recognition.listening = true
        this.listening = true
        this.speaking = false
      },
      stopRecognition(fullStop=true) {
        this.recognition.stop()
        this.recognition.listening = false
        this.listening = false
        this.speaking = !fullStop
      },
      clearMessages() {
        this.$axios.$put(`${ process.env.JSONBIN_ENDPOINT }/b/${ process.env.BIN_ID }`, [null], {
          headers: { 'Content-Type': 'application/json' }
        })
        .catch( err => {
          console.log(err)
        })
      },
      saveMessages() {
        this.$axios.$put(`${ process.env.JSONBIN_ENDPOINT }/b/${ process.env.BIN_ID }`, this.messagesList, {
          headers: { 'Content-Type': 'application/json' }
        })
        .catch( err => {
          console.log(err)
        })
      },
      async loadMessages() {
        let data = await this.$axios.$get(`${ process.env.JSONBIN_ENDPOINT }/b/${ process.env.BIN_ID }/latest`)
        // headers: { 'secret-key': process.env.JSONBIN_KEY }

        this.messagesList = data
      }
    },

  }
</script>

<style scoped>
  audio{
    position: absolute;
    opacity: 0;
    z-index: -1;
  }
  .app-container{
    height: 100%;
    padding-bottom: 52px;
    overflow: auto;
  }

  .panel, .chat-input{
    display: none;
  }
  .panel.visible, .chat-input.visible{
    display: block;
  }

  .btn img{
    height: 24px;
    width: 20px;
    margin-bottom: -5px;
  }


</style>
