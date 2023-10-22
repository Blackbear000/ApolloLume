<template>
  <div id="app">
    <div>
      <label for="real-time-option">Select Mode:</label>
      <select id="real-time-option" v-model="selectedMode" @change="handleModeChange">
        <option value="non-real-time">Non Real-Time</option>
        <option value="real-time">Real-Time</option>
      </select>
    </div>
    <div class="file-upload" v-if="selectedMode === 'non-real-time'">
      <input type="file" @change="handleFileChange">
    </div>
    <div class="images-container">
      <div class="image-container" v-if="selectedMode !== 'real-time'">
        <h2 class="title">Original</h2>
        <img v-if="originalImage" :src="originalImage" alt="Original Image">
      </div>
      <div class="image-container" v-if="selectedMode === 'real-time'">
        <h2 class="title">Real-Time Preview</h2>
        <vue-web-cam ref="video" :autoplay="true"></vue-web-cam>
      </div>
      <div class="image-container">
        <h2 class="title">ApolloLumed</h2>
        <div v-if="showProgressBar">
          <progress :value="brightnessProgress" max="100"></progress>
        </div>
        <img v-if="brightenedImage && !showProgressBar" :src="brightenedImage" alt="Brightened Image">
      </div>
    </div>
    <div v-if="selectedMode === 'real-time'">
      <p class="real-time-alert">Please connect to your Phone/Camera</p>
    </div>
  </div>
</template>

<script>
import VueWebCam from 'vue-web-cam';

export default {
  components: {
    'vue-web-cam': VueWebCam,
  },
  data() {
    return {
      originalImage: null,
      brightenedImage: null,
      selectedMode: 'non-real-time',
      showProgressBar: false,
      brightnessProgress: 0,
    };
  },
  methods: {
    handleModeChange() {
      if (this.selectedMode === 'real-time') {
        this.clearPreviousResults(); // Clear previous results when switching to real-time
        this.startRealTimePreview();
        this.originalImage = null; // Hide the Original image
      } else {
        this.stopRealTimePreview();
      }
    },
    handleFileChange(event) {
      // Handle file change based on the selected mode
      if (this.selectedMode === 'non-real-time') {
        const file = event.target.files[0];

        if (file) {
          this.originalImage = URL.createObjectURL(file);

          const reader = new FileReader();
          reader.onload = (e) => {
            const img = new Image();
            img.src = e.target.result;

            img.onload = () => {
              // 调整亮度
              this.showProgressBar = true;
              this.brightnessProgress = 0;
              this.adjustBrightness(img, 1.5)
                .then((brightenedDataUrl) => {
                  this.brightenedImage = brightenedDataUrl;
                  this.showProgressBar = false;
                });
            };
          };

          reader.readAsDataURL(file);
        }
      }
    },
    adjustBrightness(img, factor) {
      return new Promise((resolve) => {
        const canvas = document.createElement('canvas');
        const ctx = canvas.getContext('2d');

        canvas.width = img.width;
        canvas.height = img.height;

        ctx.filter = `brightness(${factor})`;
        ctx.drawImage(img, 0, 0, img.width, img.height);

        // Simulating a delay to show progress bar
        const intervalId = setInterval(() => {
          this.brightnessProgress += 5;
          if (this.brightnessProgress >= 100) {
            clearInterval(intervalId);
            resolve(canvas.toDataURL());
          }
        }, 100);
      });
    },
    startRealTimePreview() {
      navigator.mediaDevices.getUserMedia({ video: true })
        .then((stream) => {
          // Use the VueWebCam component to display the video stream
          console.log('getUserMedia success:', stream);
          this.$refs.video.startWebCam(stream);
        })
        .catch((error) => {
          console.error('Error accessing camera:', error);
        });
    },
    stopRealTimePreview() {
      // Cleanup: stop the video stream when switching from real-time mode
      this.$refs.video.stop();
    },
    clearPreviousResults() {
      this.originalImage = null;
      this.brightenedImage = null;
      this.showProgressBar = false;
      this.brightnessProgress = 0;
    },
  },
  beforeDestroy() {
    // Cleanup: stop the video stream when the component is destroyed
    this.stopRealTimePreview();
  },
};
</script>

<style scoped>
#app {
  text-align: center;
  margin-top: 50px;
}

.file-upload {
  margin-left: -350px;
  margin-bottom: 10px;
}

.images-container {
  display: flex;
  justify-content: space-between;
}

.image-container {
  flex: 1;
  text-align: center;
}

.title {
  user-select: none;
}

img {
  max-width: 100%;
  max-height: 400px;
  user-select: none;
}

.real-time-alert {
  color: red;
  font-weight: bold;
}

progress {
  width: 100%;
  margin-top: 10px;
}
</style>
