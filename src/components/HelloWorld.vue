<template>
  <div class="hello">
    <h1>{{ msg }}</h1>
    <p>
      Simple demo to demonstrate the Web Speech API using the
      <a href="https://github.com/@mastashake08/speech-kit" target="_blank" rel="noopener">SpeechKit npm package</a>!
    </p>
    <textarea v-model="voiceText" @change="generateSSML"/>
    <p>{{voiceSSML}}</p>
    <ul>
    <select name="voices" id="voice-select"  :value="selectedIndex" @change="setVoice($event)">
      <option disabled value="-1">Select Voice</option>
        <option v-for="(voice, index) in voices" :key="index" :value="index" >{{voice.name}} - {{voice.lang}}</option>
    </select>
  </ul>
    <ul>
      <button @click="share" >Share</button>
      <button @click="speak" v-if="!isPlaying">Speak</button>
      <button @click="shutup" v-else>Shut Up</button>
      <button @click="listen" v-if="!isListen">Listen</button>
      <button @click="stopListen" v-else>Stop Listen</button>
      <button @click="addBreak">Add Break</button>
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
    this.sk = new SpeechKit({continuous:true, rate: 0.85})
    setTimeout(() => {
      this.voices = this.sk.getVoices()
      this.selectedVoice = this.voices[0]
    }, "1000")
    this.generateSSML()
    document.addEventListener('onspeechkitresult', (e) =>  this.getText(e))
    document.addEventListener('onspeechkitspeechend', () =>  this.addPeriod())
    document.addEventListener('onspeechkitsoundend', () => this.addPeriod())
    document.addEventListener('onspeechkitvoiceschanged', () =>  {  this.sk.getVoices()})
  },
  data () {
    return {
      voiceText: `Welcome to the SpeechKit demo hosted on Github Pages! To get started you can select a voice from the dropdown and click the speak button to hear these instructions. You can type text or hit the listen button to enable speech recognition and change the text in the textarea.

If you click the Generate SSML button, then your text will be converted to SSML format to export in other voice-powered applications such as Amazon Alexa Polly and Twilio.

If you select a portion of a sentence you want to pause on before speaking and then click the Add Break button, your text will be converted to SSML with a 200ms break before said sentence. More features are coming to edit SSML data in the future, for now, HAVE FUN!`,
      sk: {},
      isListen: false,
      voices: [],
      selectedVoice: {},
      selectedIndex: -1,
      isPlaying: false,
      voiceSSML: null
    }
  },
  watch: {
    voiceText: function() {
      this.generateSSML()
    }
  },
  methods: {
    setVoice (e) {
      this.selectedVoice = this.voices[e.target.value]
      this.selectedIndex = e.target.value
      this.sk.setSpeechVoice(this.selectedVoice)

    },
    share () {
      const text = `Check out the SpeechKit Demo and speak this text! ${this.voiceText} ${document.URL}`
      try {
        if (!navigator.canShare) {
          this.clipBoard(text)
        } else {
          navigator.share({
            url: navigator.URL
          })
        }
      } catch (e) {
        this.clipBoard(text)
      }
    },
    async clipBoard (text, type = "text/plain") {
      const blob = new Blob([text], { type });
      const data = [new window.ClipboardItem({ [type]: blob })];
      await navigator.clipboard.write(data)
      alert ('Text copied to clipboard')
    },
    speak () {
      this.sk.speak(this.voiceSSML, this.selectedVoice)
      this.isPlaying = !this.isPlaying
    },
    shutup () {
      this.sk.shutup()
      this.isPlaying = !this.isPlaying
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
      this.voiceText = ''
      const results = evt.detail.results
      for(let i = 0; i < results.length; ++i) {
        this.voiceText += ` ${results[i][0].transcript.charAt(1).toUpperCase()}${results[i][0].transcript.slice(2)}`
        this.addPeriod()
      }
    },
    generateSSML () {
      const xml = this.sk.createSSML(this.voiceText)
      this.voiceSSML = xml
    },
    addPeriod () {
      if(this.voiceText.charAt(this.voiceText.length - 1) !== '.'){
        this.voiceText = this.voiceText + '.'
      }
    },
    addBreak () {
      let selection = window.getSelection();
      selection.modify('move', 'backward', "sentence");
      selection.modify('extend', 'forward', "sentence")
      this.voiceSSML = this.sk.addBreakSSML(this.voiceText, selection.toString(), selection.focusOffset)
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
textarea {
    width: 70%;
    height: 200px;
    -webkit-box-sizing: border-box;
       -moz-box-sizing: border-box;
            box-sizing: border-box;
}
</style>
