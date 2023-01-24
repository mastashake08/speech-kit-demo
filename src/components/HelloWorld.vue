<template>
  <div class="hello">
    <h1>{{ msg }}</h1>
    <p>
      Simple demo to demonstrate the Web Speech API using the
      <a href="https://github.com/@mastashake08/speech-kit" target="_blank" rel="noopener">SpeechKit npm package</a>!
    </p>
    <textarea v-model="voiceText" @change="generateSSML"/>
    <p>
      <span>
        <p>{{voiceSSML}}</p>
      </span>
    </p>
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
      <button @click="startStreaming">Save Audio File</button>
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
    this.mediaStream_ = new MediaStream()
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
      voiceSSML: null,
      oldVoiceSSML: '',
      SSMLTagIndicies: [],
      mediaRecorder: {},
      mediaStream_: {},
      chunks: []
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
      try {
        if (!navigator.canShare) {
          this.clipBoard(this.voiceSSML)
        } else {
          navigator.share({
            text: this.voiceSSML
          })
        }
      } catch (e) {
        this.clipBoard(this.voiceSSML)
      }
    },
    async clipBoard (text, type = "text/plain") {
      const blob = new Blob([text], { type });
      const data = [new window.ClipboardItem({ [type]: blob })];
      await navigator.clipboard.write(data)
      alert ('SSML copied to clipboard!')
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
      if(this.voiceSSML !== null) {
        this.oldVoiceSSML = this.voiceSSML
      } else {
        this.voiceSSML = this.oldVoiceSSML = xml
      }

      this.voiceSSML = xml
    },
    addPeriod () {
      if(this.voiceText.charAt(this.voiceText.length - 1) !== '.'){
        this.voiceText = this.voiceText + '.'
      }
    },
    addBreak () {
      let selection = this.getCursorSelection()
      this.voiceSSML = this.sk.addBreakSSML(this.voiceText, selection.toString(), selection.focusOffset)
    },
    addEmphasis () {
      let selection = this.getCursorSelection()
      this.voiceSSML = this.sk.addEmphasisSSML(this.voiceText, selection.toString(), selection.focusOffset)
      this.oldVoiceSSML = this.voiceSSML
    },
    getCursorSelection () {
      let selection = window.getSelection();
      selection.modify('move', 'backward', "sentence");
      selection.modify('extend', 'forward', "sentence")
      return selection
    },
    saveBlob ()  {
      const a = document.createElement('a');
      document.body.appendChild(a);
      a.style.display = 'none';
      const blob = new Blob(this.chunks)
      const filename = `${Date.now()}.webm`
      const url = URL.createObjectURL(blob);
      a.href = url;
      a.download = filename;
      a.click()
    },
    async startStreaming () {
      const stream = await navigator.mediaDevices.getDisplayMedia({
        video: true,
        audio: true
      })
      this.mediaRecorder = new MediaRecorder(stream)
      this.isPlaying = ! this.isPlaying
      const track = await stream.getAudioTracks()[0]
      document.addEventListener('onspeechkitutterenceend', () => {
        this.mediaRecorder.stop()
      })
      this.mediaStream_.addTrack(track)
      this.mediaRecorder.ondataavailable = event => {
        if (event.data.size > 0) {
          this.chunks.push(event.data);
        }
      }
      this.mediaRecorder.onstop = () => {
        this.sk.shutup()
        track.stop();
        this.mediaStream_.getAudioTracks()[0].stop();
        this.mediaStream_.removeTrack(track);
        this.saveBlob()
      }
      this.mediaRecorder.start()
      this.speak()
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
span {
    width: 100%;
    max-width: 80%;
    text-align: center;
}
span > p {
  border: 1px solid red;
}
</style>
