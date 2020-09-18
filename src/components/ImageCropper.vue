<template>
  <div id="main-container">
    <img class="brand-logo" src="../assets/goldcast-logo.svg" alt />
    <div class="image-cropper-page" @drop.stop.prevent>
      <ImageDrop
        @dragend="handleDragEnd"
        @dropped="handleImageDrop"
        @selectorMoved="handleSelectorMove"
      />
      <div id="cropped-image-container">
        <canvas width="500" height="500" ref="croppedCanvas" class="cropped-image-canvas"></canvas>
      </div>
    </div>
    <div class="buttons-container">
      <button
        class="ripple"
        :class="{'disabled': !imageSource}"
        :disabled="!imageSource"
        v-on:click="getCroppedImage"
      >CROP IMAGE</button>
      <a
        class="ripple"
        :class="{'disabled': !isImageCropped}"
        @click.stop
        :href="downloadLink"
        download="myImage.jpg"
        v-on:click="downloadCroppedImage"
      >
        <span>DOWNLOAD RESULT</span>
      </a>
    </div>
  </div>
</template>

<script>
import ImageDrop from "./ImageDrop.vue";
export default {
  name: "ImageCropper",
  components: {
    ImageDrop,
  },
  props: {
    msg: String,
  },
  data() {
    return {
      downloadLink: "",
      isImageCropped: false,
      croppedImageNode: null,
      imageSource: null,
      cropCoordinates: {
        startX: null,
        startY: null,
      },
    };
  },
  methods: {
    downloadCroppedImage() {
      let canvas = this.$refs.croppedCanvas;
      let image = canvas.toDataURL("image/jpg");
      this.downloadLink = image;
    },
    handleDragEnd(coordinates) {
      this.cropCoordinates = coordinates;
    },
    handleSelectorMove(coordinates) {
      let vm = this;
      let image = new Image();
      let croppedImage = new Image();
      image.src = this.imageSource;
      image.onload = function () {
        let croppedImageCanvas = vm.cropImage(image, {
          x: (coordinates.offsetX - 50) * 4.8, // crop keeping the center
          y: (coordinates.offsetY - 50) * 4.8,
          width: this.width,
          height: this.height,
        });
        croppedImage.style = `width: 104px; height: 104px;`;
        croppedImage.src = croppedImageCanvas.toDataURL();
      };

      croppedImage.onload = function () {
        document.getElementById("cropped-image-container").appendChild(this); // Add the image to the DOM
      };
    },
    getCroppedImage() {
      console.log("getCroppedImage");
      const vm = this;
      let image = new Image();
      image.src = this.imageSource;
      image.onload = function () {
        vm.cropImage(this, {
          x: vm.cropCoordinates.startX, // crop keeping the center
          y: vm.cropCoordinates.startY,
          width: this.width,
          height: this.height,
        });
      };
    },
    handleImageDrop(imageSource) {
      this.imageSource = imageSource;
    },
    cropImage(image, croppingCoords) {
      let cc = croppingCoords;
      let croppedImageWidth = cc.width / 4;
      let workCan = this.$refs.croppedCanvas;
      workCan.width = croppedImageWidth; // set the canvas resolution to the cropped image size
      workCan.height = croppedImageWidth;
      let ctx = workCan.getContext("2d"); // get a 2D rendering interface
      let clipStartX = cc.x ? cc.x : image.width / 2 - image.width / 8;
      let clipStartY = cc.y ? cc.y : image.height / 2 - image.width / 8;

      // console.log(image, startX, startY, cc.width, cc.height, startX, startY, croppedImageWidth,  croppedImageWidth);
      ctx.drawImage(
        image,
        clipStartX,
        clipStartY,
        workCan.width,
        workCan.height,
        0,
        0,
        croppedImageWidth,
        croppedImageWidth
      ); // draw the image offset to place it correctly on the cropped region
      this.isImageCropped = true;
    },
  },
};
</script>

<style scoped>
/* TO DO after uploading coding challenge: replace for SCSS files, add a SASS loader, responsiveness :) */

#main-container {
  width: 100%;
}

.brand-logo {
  position: fixed;
  top: 24px;
  left: 36px;
}

.image-cropper-page {
  width: 1400px;
  margin: auto;
  height: 100%;
  display: flex;
  justify-content: center;
}

button,
a {
  color: #fff;
  background-color: rgba(0, 217, 177, 1);
  box-shadow: 0px 3px 1px -2px rgba(0, 0, 0, 0.2),
    0px 2px 2px 0px rgba(0, 0, 0, 0.14), 0px 1px 5px 0px rgba(0, 0, 0, 0.12);
  padding: 6px 16px;
  border: none;
  cursor: pointer;
  margin: 0 8px 0 8px;
  display: inline-flex;
  outline: 0;
  position: relative;
  align-items: center;
  font-weight: bold;
  font-family: Roboto;
  line-height: 1.75;
  border-radius: 4px;
  letter-spacing: 0.02857em;
  font-size: 14px;
}

span {
  text-transform: capitalize;
}

a {
  text-decoration: none;
}

.buttons-container {
  width: 1400px;
  margin: auto;
  display: flex;
  justify-content: center;
  align-items: center;
  padding-top: 32px;
  padding-bottom: 32px;
}

.disabled {
  pointer-events: none;
  color: rgba(0, 0, 0, 0.26);
  box-shadow: none;
  background-color: #f9f9f9;
}

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

#cropped-image-container {
  width: 500px;
  height: 500px;
  padding: 1rem;
  /* border: 10px solid #eee; */
  border-radius: 4px;
  background-color: #eee;
  margin-left: 24px;
  align-items: center;
  justify-content: center;
  display: flex;
}

/* Ripple effect */
.ripple {
  background-position: center;
  transition: background 0.8s;
}
.ripple:hover {
  background: rgba(0, 217, 177, 0.8)
    radial-gradient(circle, transparent 1%, rgba(0, 217, 177, 0.8) 1%)
    center/15000%;
}
.ripple:active {
  background-color: rgba(0, 217, 177, 0.6);
  background-size: 100%;
  transition: background 0s;
}

#box {
  border-top: 25px solid transparent;
  border-right: 50px solid #555;
  border-bottom: 25px solid transparent;
  position: absolute;
  top: 0;
  left: 0;
}
</style>
