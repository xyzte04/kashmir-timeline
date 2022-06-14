<template>
  <div class="main-container">
    <Headline />
    <div data-sound="off" id="sound-icon">
      <img class="sound-on" src="images/sound-on.svg" />
      <img class="sound-off" src="images/sound-off.svg" />
      <audio id="audio" src="files/Melancholia - Godmode.mp3"></audio>
      <span class="sound-label">Toggle sound</span>
    </div>
    <div class="map-scrolly-container">
      <div class="back-img"></div>
      <MapTimeLine />
      <Scroller />
    </div>
  </div>
</template>

<script>
import * as d3 from "d3";
import MapTimeLine from "./components/MapTimeline.vue";
import Scroller from "./components/Scroller.vue";
import Headline from "./components/Headline.vue";

export default {
  name: "App",
  components: {
    MapTimeLine,
    Scroller,
    Headline
  },
  mounted: function() {
    window.audioPlaying = false;
    const audio = document.getElementById("audio");
    audio.pause();

    d3.select("#sound-icon").on("click", function() {
      if (!window.audioPlaying) {
        audio.play();
        window.audioPlaying = true;
        document.getElementById("sound-icon").dataset.sound = "on";
      } else {
        audio.pause();
        window.audioPlaying = false;
        document.getElementById("sound-icon").dataset.sound = "off";
      }
    });
  }
};
</script>

<style lang="scss">
body {
  padding: 0px;
  margin: 0px;
  background-color: #3a3a3a;
  font-family: "Open Sans", sans-serif;
  overflow-x: hidden;
  * {
    max-width: 100vw;
  }
}

* {
  box-sizing: border-box;
}

.map-scrolly-container {
  position: relative;
  .back-img {
    background-image: url("../public/images/collage.jpg");
    background-size: cover;
    height: 100vh;
    width: 100vw;
    background-repeat: no-repeat;
    position: sticky;
    top: 0;
  }
}

#sound-icon {
  position: fixed;
  top: 10px;
  right: 10px;
  z-index: 100000;
  text-align: center;
  .sound-label {
    font-size: 0.6rem;
    font-style: italic;
    color: white;
    position: relative;
    bottom: 6px;
  }
  img {
    width: 30px;
    margin: auto;
  }
  &[data-sound="on"] {
    img.sound-on {
      display: block;
    }
    img.sound-off {
      display: none;
    }
  }
  &[data-sound="off"] {
    img.sound-on {
      display: none;
    }
    img.sound-off {
      display: block;
    }
  }
}
</style>
