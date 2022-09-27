<template>
  <glassPane text="Alan Way"></glassPane>
</template>

<script>
import HelloWorld from './components/HelloWorld.vue'
import glassPane from './components/glassPane.vue'
import shardCanvas from './components/shardCanvas.vue'

export default {
  name: 'App',
  components: {
    HelloWorld,
    glassPane,
    shardCanvas
  }
}
</script>


<style lang="scss">
  :root {
    --matrix-grey: #272727;
    // --matrix-grey: #343434;
    --sunken-grey: #141414;
    --melancholy-red: #b7482f;
    --florea: #0f4785;
    --cvijet: #041e3b;
    --somber-umber: #5d0505;
    --soft-white: #f3f3f3;
    --soft-white-T: rgba(159, 159, 159, 0.31);
    --clear-glass: rgba(156, 209, 220, 0.2);
    --clear-glass-light: rgba(0, 215, 255, 0.1);
    --glass-undertone: rgba(111, 206, 255, 0.13);
    --glass-edge-white: rgba(255, 255, 255, 0.33);
  
    // Crack shadow variables.
    --crack-shadow-horizontal-offset: 4px;
    --crack-shadow-vertical-offset: 4px;
    --crack-shadow-spread: 2px;
  
    // font-family: 'Cinzel Decorative', cursive;
    // font-family: 'Roboto', sans-serif;
    --crack-path: polygon(
      -400% -400%,
      500% 0%,
      500% 500%,
      -400% 500%,
      45.4% 63.8%,
      46.2% 63%,
      46.4% 63.2%,
      46.6% 64.2%,
      47% 64.4%,
      46.8% 66.5%,
      47.35% 66.2%,
      47.48% 65.2%,
      48.2% 65.2%,
      48.4% 64.2%,
      49.9% 62.8%,
      50% 61.8%,
      50.2% 61.2%,
      50.25% 60.8%,
      49.4% 60.3%,
      48.85% 59.7%,
      48.5% 61%,
      47.7% 60.6%,
      46.9% 60.8%,
      45.9% 56.8%,
      45.2% 53.5%,
      44.7% 53.3%,
      43.4% 54.33%,
      43% 54.65%,
      42.6% 54.2%,
      41.7% 55%,
      43.45% 57.1%,
      43.45% 57.8%,
      42.9% 58.9%,
      43.2% 59.2%,
      43.76% 61.3%,
      43.9% 61.5%,
      43.95% 62.2%,
      44.3% 63.2%,
      44.5% 63.4%,
      45.4% 65%,
      -400% 500%
    );
  }

  body {
    background-color: #434343;
    margin: 0;
  }

  // https://stackoverflow.com/questions/826782/how-to-disable-text-selection-highlighting
  .noselect {
    -webkit-touch-callout: none; /* iOS Safari */
      -webkit-user-select: none; /* Safari */
       -khtml-user-select: none; /* Konqueror HTML */
         -moz-user-select: none; /* Old versions of Firefox */
          -ms-user-select: none; /* Internet Explorer/Edge */
              user-select: none; /* Non-prefixed version, currently
                                    supported by Chrome, Edge, Opera and Firefox */
  }
</style>