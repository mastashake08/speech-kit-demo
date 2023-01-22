<template>
  <div class="hello">
    <h1>{{ msg }}</h1>
    <p>
      Simple demo to demonstrate the Web Speech API using the
      <a href="https://github.com/@mastashake08/speech-kit" target="_blank" rel="noopener">SpeechKit npm package</a>!
    </p>
    <textarea v-model="voiceText"/>
    <ul>
      <button @click="share" >Share</button>
      <button @click="speak">Speak</button>
      <button @click="listen" v-if="!isListen">Listen</button>
      <button @click="stopListen" v-else>Stop Listen</button>
    </ul>
  </div>
</template>

<script>
import SpeechKit from '@mastashake08/speech-kit'
export default {
  name: 'HelloWorld',
  props: {
    msg: String
  },
  mounted () {
    this.sk = new SpeechKit({rate: 0.85})
    document.addEventListener('onspeechkitresult', (e) => this.getText(e))
  },
  data () {
    return {
      voiceText: 'SPEAK ME',
      sk: {},
      isListen: false
    }
  },
  methods: {
    share () {
      const text = `Check out the SpeechKit Demo and speak this text! ${this.voiceText} ${document.URL}`
      try {
        if (!navigator.canShare) {
          this.clipBoard(text)
        } else {
          navigator.share({
            text: text,
            url: 'https://speechkit.jcompsolu.com'
          })
        }
      } catch (e) {
        this.clipBoard(text)
      }
    },
    async clipBoard (text) {
      const type = "text/plain";
      const blob = new Blob([text], { type });

      const data = [new window.ClipboardItem({ [type]: blob })];
      await navigator.clipboard.write(data)
      alert ('Text copied to clipboard')
    },
    speak () {
      this.sk.speak(this.voiceText)
    },
    listen () {
      this.sk.listen()
      this.isListen = !this.isListen
    },
    stopListen () {
      this.sk.stopListen()
      this.isListen = !this.isListen
    },
    getText (evt) {
      this.voiceText = evt.detail.transcript
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
