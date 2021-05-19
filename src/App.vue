<template>
  <div id="app">
    <h1></h1>
    
    <GCodePreview ref="gcodePreview1" class="gcode-preview"
      :startLayer=20
      :endLayer=150
      :topLayerColor="'red'"
      :lastSegmentColor="'red'"
    />

    <!-- <div># layers loaded: {{ layersLoaded }}</div> -->
  </div>
</template>

<script>
 

import GCodePreview from './components/GCodePreview.vue';
let layersLoaded = 0;
const chunkSize = 250;
export default {
  components: {
    GCodePreview
  },

  data() {
    return {
      layersLoaded : layersLoaded
    }
  },

  async mounted() {
    
    const lines1 = await this.fetchGcode('/cube.gcode');
    //const lines1 = 'G0 X0 Y0 Z0.2\nG1 X42 Y42';
    this.loadPreviewChunked(this.$refs.gcodePreview1, lines1, 500);

    const lines2 = await this.fetchGcode('/duplo_tracks.gcode');
    this.loadPreviewChunked(this.$refs.gcodePreview2, lines2, 50);
  },

  methods : {

    async fetchGcode(url) {
      const response = await fetch(url);

      if (response.status !== 200) {
        throw new Error(`status code: ${response.status}`);
      }

      const file = await response.text();
      return file.split('\n');
    },
    
    loadPreviewChunked(target, lines, delay) {
      let c = 0;
      const id = '__animationTimer__' + Math.random().toString(36).substr(2, 9);
      const loadProgressive = (preview) => {
        const start = c*chunkSize;
        const end = (c+1)*chunkSize;
        const chunk = lines.slice(start, end);
        target.processGCode(chunk)
        // this.layersLoaded = target.layerCount;
        c++;
        if (c*chunkSize < lines.length) { 
          window[id] = setTimeout(loadProgressive, delay);
        }
      }
      // cancel loading process if one is still in progress
      // mostly when hot reloading
      window.clearTimeout(window[id]);
      loadProgressive(target);
    }
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

.gcode-preview {
  width: 80vw;
  margin: auto;
}
</style>
