<template>
  <div class="h-[100dvh] flex flex-col ">
    <!-- Top Section (Title) -->
    <header class="h-[15%] flex flex-col items-center justify-center bg-blue-500 text-white text-2xl font-bold">
        <h1 class="w-full text-center">{{ currentYear }}</h1>
        <h3 class="text-sm">今年の恵方巻きの方角は：<strong>{{ currentDetail?.direction }} / {{ currentDetail?.degrees }}°</strong></h3>
        <h4>You facing: {{ deviceRotation.toFixed(1) }}°</h4>
        <!-- <h5>Message: {{ message }}</h5> -->
    </header>


    <!-- Center Section (Main Content) -->
    <main class="h-[60%] flex items-center justify-center bg-white">
      <div class="compass">
          <div class="body">
              <div class="take"></div>
              <div class="panel">
                  <div class="hold-bg" :style="{ transform: `rotate(${deviceRotation}deg)` }">
                      <div class="glass"></div>
                      <div class="hold-mark">
                          <div v-for="row in 6" :key="row">
                            <span v-for="col in 4" :key="col"></span>
                          </div>
                      </div>
                      <div class="hold-arrows">
                          <div class="arrow arrow-up"></div>
                          <div class="arrow arrow-right"></div>
                          <div class="arrow arrow-down"></div>
                          <div class="arrow arrow-left"></div>
                          <div class="arrow-sub arrow-up-right"></div>
                          <div class="arrow-sub arrow-up-left"></div>
                          <div class="arrow-sub arrow-down-right"></div>
                          <div class="arrow-sub arrow-down-left"></div>
                      </div>
                      <div class="hold-directions">
                          <div class="direction direction-n">北</div>
                          <div class="direction direction-l">東</div>
                          <div class="direction direction-s">南</div>
                          <div class="direction direction-o">西</div>
                          <div class="direction-sub direction-ne">北東</div>
                          <div class="direction-sub direction-no">北西</div>
                          <div class="direction-sub direction-se">南東</div>
                          <div class="direction-sub direction-so">南西</div>
                      </div>
                  </div>
                  <div class="hold-main-arrow">
                      <div class="main-arrow main-arrow-pointer"></div>
                      <div class="main-arrow down"></div>
                  </div>
                  <div class="center"></div>
              </div>
          </div>
      </div>
    </main>

    <!-- Bottom Section (Controller) -->
    <footer class="h-[25%] flex flex-col items-center justify-center bg-gray-800 text-white">
        Controller Section <br>
        <button class="bg-gray-200" v-if="!permissionGranted" @click="requestPermission">
          Enable Motion Sensors
        </button>
    </footer>

  </div>
  
</template>

<script>

export default {
  name: 'App',
  components: {
  },
  data(){
    return{
      currentYear: null,
      currentDirection: null,
      currentDetail: null,

      // deviceRotation: 0,
      deviceRotation: 0,

      message: null,
      permissionGranted: false,

    }
  },
  methods: {
    getDirection(){
      this.currentDirection = 'hey'

      const lastDigit = this.currentYear % 10; 
      let direction, exactDirection, degrees, japaneseName;

      switch (lastDigit) {
        case 0:
        case 5:
            direction = "西南西";
            exactDirection = "西南西微西 (Sei-Nan-Sei-Bi-Sei) / Slightly More West";
            degrees = 255;
            japaneseName = "戊 (Tsuchinoe, つちのえ)";
            break;
        case 1:
        case 3:
        case 6:
        case 8:
            direction = "南南東";
            exactDirection = "南南東微南 (Nan-Nan-Tō-Bi-Nan) / Slightly More South";
            degrees = 165;
            japaneseName = "丙 (Hinoe, ひのえ)";
            break;
        case 2:
        case 7:
            direction = "北北西";
            exactDirection = "北北西微北 (Hoku-Hoku-Sei-Bi-Hoku) / Slightly More North";
            degrees = 345;
            japaneseName = "壬 (Mizunoe, みずのえ)";
            break;
        case 4:
        case 9:
            direction = "東北東";
            exactDirection = "東北東微東 (Tō-Hoku-Tō-Bi-Tō) / Slightly More East";
            degrees = 75;
            japaneseName = "庚 (Kanoto, かのえ)";
            break;
        default:
            console.log("Invalid year");
      }

      this.currentDetail = {
        direction,
        exactDirection,
        degrees,
        japaneseName,
      }

    },
    requestPermission() {
      if (typeof DeviceOrientationEvent.requestPermission === "function") {
        DeviceOrientationEvent.requestPermission()
          .then((permissionState) => {
            if (permissionState === "granted") {
              this.message = "Permission granted!";
              window.addEventListener("deviceorientation", this.handleOrientation);
            } else {
              this.message = "Permission denied";
            }
          })
          .catch((err) => {
            this.message = `Error: ${err}`;
          });
      } else {
        this.message = "Permission request not needed";
        window.addEventListener("deviceorientation", this.handleOrientation);
      }
    },
    handleOrientation(event) {
      if (event.alpha !== null) {
        this.deviceRotation = event.alpha;
        this.message = `Rotating: ${this.deviceRotation}°`;
      } else {
        this.message = "Device orientation not supported";
      }
    },


  },

  mounted() {
  console.clear();
  this.message = "Waiting for motion permission...";

  // Check if DeviceOrientationEvent.requestPermission is needed (iOS case)
  if (typeof DeviceOrientationEvent.requestPermission === "function") {
    this.message = "Tap the button to enable motion sensors.";

    // Do not request permission here directly, instead rely on user interaction
  } else {
    // If permission is not required (Android, some browsers), attach the event listener directly
    this.message = "Permission not required, attaching event listener.";
    window.addEventListener("deviceorientation", this.handleOrientation);
  }

  // Get the current year and determine direction
  this.currentYear = new Date().getFullYear();
  this.getDirection();
},


  beforeUnmount() {
    window.removeEventListener("deviceorientation", this.handleOrientation);
  },
}
</script>

<style>
body.device-mobile-optimized>*{
    max-width: unset !important;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  text-align: center;
  color: #2c3e50;

  margin: 0;
  padding: 0;
  height: 100%;
  transition: all .25s ease-in-out;

  overflow: hidden;
  
}


.compass {
    position: relative;
    width: 85dvw;
    aspect-ratio: 1;
    /* margin: 50px auto 0; */

    max-width: 305px;
}

.body {
    position: absolute;
    z-index: 1;
    /* left: 25px;
    top: 100px; */
    width: 100%;
    height: 100%;
    border-radius: 50%;
}

.body:after {
    position: absolute;
    content: '';
    width: 105%;
    aspect-ratio: 1;
    /* height: 670px; */
    display: block;
    top: 50%;
    left: 50%;
    transform: translate(-50%,-50%);
    /* left: -10px;
    top: -10px; */
    z-index: -1;
    border-radius: 50%;
    background-color: #c8603d;
    border: 2px solid #763c24;
    box-shadow:
        0 0px 0px 8px #9b472b,
        0 10px 0px 8px #743621;
}

/*  Painel
========================================================================== */
.panel {
position: absolute;
background-color: #252b29;
width: 100%;
height: 100%;
border-radius: 50%;
overflow: hidden;
}

/*  Centro
========================================================================== */
.center {
    position: absolute;
    z-index: 20;
    width: 25px;
    aspect-ratio: 1;
    left: 50%;
    top: 50%;
    /* margin: -20px 0 0 -22px; */
    border-radius: 50%;

    background: rgb(240,183,161);
    background:
        -moz-linear-gradient(-45deg,
            rgba(240,183,161,1) 0%,
            rgba(140,51,16,1) 50%,
            rgba(117,34,1,1) 51%,
            rgba(191,110,78,1) 100%);
    background: 
        -webkit-gradient(linear, left top, right bottom, 
            color-stop(0%,rgba(240,183,161,1)),
            color-stop(50%,rgba(140,51,16,1)), 
            color-stop(51%,rgba(117,34,1,1)), 
            color-stop(100%,rgba(191,110,78,1)));
    background:
        -webkit-linear-gradient(-45deg,  
            rgba(240,183,161,1) 0%,
            rgba(140,51,16,1) 50%,
            rgba(117,34,1,1) 51%,
            rgba(191,110,78,1) 100%);
    background:
        -o-linear-gradient(-45deg,
            rgba(240,183,161,1) 0%,
            rgba(140,51,16,1) 50%,
            rgba(117,34,1,1) 51%,
            rgba(191,110,78,1) 100%);
    background:
        -ms-linear-gradient(-45deg,  
            rgba(240,183,161,1) 0%,
            rgba(140,51,16,1) 50%,
            rgba(117,34,1,1) 51%,
            rgba(191,110,78,1) 100%);
    background:
        linear-gradient(135deg, 
            rgba(240,183,161,1) 0%,
            rgba(140,51,16,1) 50%,
            rgba(117,34,1,1) 51%,
            rgba(191,110,78,1) 100%);

    box-shadow: 0px 0px 10px 0px rgba(0,0,0, .5);

    transform: translate(-50%,-50%);
    pointer-events: none;
}

.center:before {
    position: absolute;
    z-index: 1;
    content: '';
    width: 10px;
    aspect-ratio: 1;
    top: 50%;
    left: 50%;
    transform: translate(-50%,-50%);
    border-radius: 50%;
    background-color: #ddd;
}

/*  Marcadores
========================================================================== */
.hold-mark {
    position: absolute;
    top: 50%;
    left: 50%;
    width: 100%;
    aspect-ratio: 1;

    transform: translate(-50%,-50%);
    
    /* margin: -310px 0 0 -310px; */
    z-index: 0;
    border-radius: 50%;

    pointer-events: none;
}

.hold-mark > div {
    border: solid 0 #9a8770;
    border-width: 10px 0;
    width: 6px;
    left: 50%;
    transform: translateX(-50%);
    /* margin-left: -4px; */
    /* top: -15px; */
    opacity: 0.3;
}

.hold-mark > div > span {
    width: 1px;
    background-color: #9a8770;
    opacity: 0.3;
}

.hold-mark > div,
.hold-mark > div > span {
    height: 100%;
    position: absolute;
    display: block;
}

.hold-mark > div:nth-child(2) {
    -webkit-transform: rotate(30deg);
        -moz-transform: rotate(30deg);
          -o-transform: rotate(30deg);
        -ms-transform: rotate(30deg);
            transform: rotate(30deg);
}

.hold-mark > div:nth-child(3) {
    -webkit-transform: rotate(60deg);
        -moz-transform: rotate(60deg);
          -o-transform: rotate(60deg);
        -ms-transform: rotate(60deg);
            transform: rotate(60deg);
}

.hold-mark > div:nth-child(4) {
    -webkit-transform: rotate(90deg);
        -moz-transform: rotate(90deg);
          -o-transform: rotate(90deg);
        -ms-transform: rotate(90deg);
            transform: rotate(90deg);
}

.hold-mark > div:nth-child(5) {
    -webkit-transform: rotate(120deg);
        -moz-transform: rotate(120deg);
          -o-transform: rotate(120deg);
        -ms-transform: rotate(120deg);
            transform: rotate(120deg);
}

.hold-mark > div:nth-child(6) {
    -webkit-transform: rotate(150deg);
        -moz-transform: rotate(150deg);
          -o-transform: rotate(150deg);
        -ms-transform: rotate(150deg);
            transform: rotate(150deg);
}

.hold-mark > div > span {
    -webkit-transform: rotate(6deg);
        -moz-transform: rotate(6deg);
          -o-transform: rotate(6deg);
        -ms-transform: rotate(6deg);
            transform: rotate(6deg);
}

.hold-mark > div > span:nth-child(1) {
    -webkit-transform: rotate(12deg);
        -moz-transform: rotate(12deg);
          -o-transform: rotate(12deg);
        -ms-transform: rotate(12deg);
            transform: rotate(12deg);
}

.hold-mark > div > span:nth-child(2) {
    -webkit-transform: rotate(18deg);
        -moz-transform: rotate(18deg);
          -o-transform: rotate(18deg);
        -ms-transform: rotate(18deg);
            transform: rotate(18deg);
}

.hold-mark > div > span:nth-child(3) {
    -webkit-transform: rotate(24deg);
        -moz-transform: rotate(24deg);
          -o-transform: rotate(24deg);
        -ms-transform: rotate(24deg);
            transform: rotate(24deg);
}


/*  BG
========================================================================== */
.hold-bg {
    /* transition: all 1s ease-in-out; */

    position: absolute;
    width: 100%;
    height: 100%;
    background:
        -moz-linear-gradient(left, 
            rgba(34,34,34,1) 0%,
            rgba(0,0,0, 0) 100%);            
    background: 
        -webkit-gradient(linear, left top, right top,
            color-stop(0%,rgba(34,34,34,1)),
            color-stop(100%,rgba(0,0,0, 0)));
    background:
        -webkit-linear-gradient(left,  
            rgba(34,34,34,1) 0%,
            rgba(0,0,0, 0) 100%);
    background:
        -o-linear-gradient(left,  
            rgba(34,34,34,1) 0%,
            rgba(0,0,0, 0) 100%);
    background: 
        -ms-linear-gradient(left,  
            rgba(34,34,34,1) 0%,
            rgba(0,0,0, 0) 100%);
    background:
        linear-gradient(to right,  
            rgba(34,34,34,1) 0%,
            rgba(0,0,0, 0) 100%);
}

.glass {
  pointer-events: none;
    position: absolute;
    width: 100%;
    height: 100%;
    z-index: 10;
    border: 1px solid rgba(255, 255, 255, 0.1);
    border-bottom: 0;
    background: 
        -moz-linear-gradient(-45deg, 
            rgba(255,255,255,0) 0%, 
            rgba(255,255,255,0.15) 50%, 
            rgba(255,255,255,0) 50%, 
            rgba(255,255,255,0) 100%);
    background: 
        -webkit-gradient(linear, left top, right bottom, 
            color-stop(0%,rgba(255,255,255,0)), 
            color-stop(50%,rgba(255,255,255,0.15)), 
            color-stop(50%,rgba(255,255,255,0)), 
            color-stop(100%,rgba(255,255,255,0)));
    background:
        -webkit-linear-gradient(-45deg, 
            rgba(255,255,255,0) 0%,
            rgba(255,255,255,0.15) 50%,
            rgba(255,255,255,0) 50%,
            rgba(255,255,255,0) 100%);
    background:
        -o-linear-gradient(-45deg, 
            rgba(255,255,255,0) 0%,
            rgba(255,255,255,0.15) 50%,
            rgba(255,255,255,0) 50%,
            rgba(255,255,255,0) 100%);
    background:
        -ms-linear-gradient(-45deg, 
            rgba(255,255,255,0) 0%,
            rgba(255,255,255,0.15) 50%,
            rgba(255,255,255,0) 50%,
            rgba(255,255,255,0) 100%);
    background:
        linear-gradient(135deg, 
            rgba(255,255,255,0) 0%,
            rgba(255,255,255,0.15) 50%,
            rgba(255,255,255,0) 50%,
            rgba(255,255,255,0) 100%);
}

/*  Setas (rosa dos ventos)
    ========================================================================== */
    .hold-arrows {
        position: absolute;
        z-index: 0;

        width: 100%;
        aspect-ratio: 1;

        left: 50%;
        top: 50%;

        transform: translate(-50%,-50%);

        pointer-events: none;
    }

    .arrow {
        position: absolute;
        z-index: 2;
        width: 0; 
        height: 0;
        top: 20%;
        left: 50%;                                
        border-left: 15px solid transparent;
        border-right: 15px solid transparent;
        border-bottom: 92px solid #9a8770;

        transform-origin: bottom left;

        transform: rotate(0deg) translateX(-50%);

        pointer-events: none;
    }

    .arrow:after {
        position: absolute;
        z-index: 0;
        content: '';
        top: 15px;
        left: -10px;
        border-left: 10px solid transparent;
        border-right: 10px solid transparent;
        border-bottom: 85px solid #252b29;
    }

    .arrow-right {transform: rotate(90deg) translateX(-50%); z-index: -1;}
    .arrow-down {transform: rotate(180deg) translateX(-50%);}
    .arrow-left {transform: rotate(270deg) translateX(-50%); z-index: -1;}

    .arrow-sub {
        position: absolute;
        z-index: -2;
        
        top: 22.5%;
        left: 50%;

        border-left: 15px solid transparent;
        border-right: 15px solid transparent;
        border-bottom: 85px solid #5c4b41;

        transform-origin: bottom;

    }

    .arrow-up-right {transform: translatex(-50%) rotate(45deg);}
    .arrow-up-left {transform: translatex(-50%) rotate(-45deg);}
    .arrow-down-left {transform: translatex(-50%) rotate(-135deg);}
    .arrow-down-right {transform: translatex(-50%) rotate(135deg);}


/*  Direções
    ========================================================================== */
    .hold-directions {
        position: absolute;
        width: 100%;
        aspect-ratio: 1;
        left: 50%;
        top: 50%;

        transform: translate(-50%,-50%);
        /* margin: -300px 0 0 -300px; */

        pointer-events: none;
    }

    .direction,
    .direction-sub {
        position: absolute;
        z-index: 2;                
        color: #706257;
        font-weight: bold;
        font-family: "Times New Roman", Times, Baskerville, Georgia, serif;
        font-size: 2em;
        text-shadow: 1px 1px 1px #000;
    }

    .direction-sub { font-size: 1.25em; }

    .direction-n { 
        top: 5%;
        left: 50%;
        transform: translateX(-50%);
    }

    .direction-l {
        right: 7.5%;
        top: 50%;
        transform: translateY(-50%) rotate(90deg);
    }

    .direction-o {
        left: 7.5%;
        top: 50%;
        transform: translateY(-50%) rotate(270deg);            
    }

    .direction-s {
        bottom: 5%;
        left: 50%;
        transform: translateX(-50%) rotate(180deg);
    }

    /* --------------------------------------------------------------------- */

    .direction-ne {
        top: 20%;
        right: 18%;
        transform: rotate(45deg);
    }

    .direction-no {
        top: 20%;
        left: 18%;
        transform: rotate(-45deg);
    }

    .direction-se {
        bottom: 20%;
        right: 18%;
        transform: rotate(135deg);
    }

    .direction-so {
        bottom: 20%;
        left: 18%;
        transform: rotate(-135deg);
    }

/*  Seta principal
    ========================================================================== */
    .hold-main-arrow {
        position: absolute;
        z-index: 10;
        width: 95%;
        aspect-ratio: 1;
        left: 50%;
        top: 50%;

        transform: translate(-50%,-50%);

        pointer-events: none;
    }

    .main-arrow {
        position: absolute;
        width: 0; 
        height: 0;
        bottom: 50%;
        left: 50%;

        border-left: 15px solid transparent;
        border-right: 15px solid transparent;
        border-bottom: 140px solid #d7dee3;
        transform-origin: bottom;

        transform: translateX(-50%);
    }

    .main-arrow:before {
        position: absolute;
        content: '';
        left: 0;
        border-left: 0 solid transparent;
        border-right: 15px solid transparent;
        border-bottom: 140px solid #f6f8f9; 
    }

    .main-arrow-pointer:after {
        position: absolute;
        top: 0;
        left: 50%;
        transform: translateX(-50%);

        content: '';
        border-left: 7px solid transparent;
        border-right: 7px solid transparent;
        border-bottom: 64px solid #d9363b;
    }

    .main-arrow.down {transform: translateX(-50%) rotate(180deg);}


</style>
