<template>
  <div
    class="drop"
    :class="getClasses"
    @dragover.prevent="dragOver"
    @dragleave.prevent="dragLeave"
    @drop.prevent="drop($event)"
  >
    <div class="dropArea" id="canvas-container">
      <canvas
        width="500"
        height="500"
        ref="canvas"
        class="drop-image-canvas"
        :class="{'dragged': isDragging}"
        v-on:mousedown="handleMouseDown"
        v-on:mousemove="handleMouseMove"
        v-on:mouseup="handleMouseOut"
        v-on:mouseout="handleMouseOut"
      ></canvas>
      <h1
        class="invalid-file-text"
        v-if="wrongFile && !imageSource"
      >File not supported, please drop an image</h1>
      <h1 class="drop-image-text" v-if="!imageSource && !isDragging && !wrongFile">Drop an image</h1>
    </div>
  </div>
</template>



<script>
export default {
  name: "ImageDrop",
  data() {
    return {
      newWidth: null,
      newHeight: null,
      startX: null,
      startY: null,
      canvasOffset: null,
      offsetX: null,
      offsetY: null,
      canvasWidth: null,
      canvasHeight: null,
      ctx: null,
      image: null,
      canvasMouseX: null,
      canvasMouseY: null,
      isDragging: false,
      wrongFile: false,
      imageSource: null,
    };
  },
  computed: {
    getClasses() {
      return { isDragging: this.isDragging };
    },
  },
  methods: {
    dragOver() {
      this.isDragging = true;
    },
    dragLeave() {
      this.isDragging = false;
    },
    drop(e) {
      let files = e.dataTransfer.files;
      this.wrongFile = false;
      // Allows a single file to be dropped
      if (files.length === 1) {
        let file = files[0];
        // Only allow images to be dropped
        if (file.type.indexOf("image/") >= 0) {
          let reader = new FileReader();
          reader.onload = (f) => {
            this.imageSource = f.target.result;
            this.drawImage(f.target.result);
            this.isDragging = false;
          };
          reader.readAsDataURL(file);
        } else {
          this.wrongFile = true;
          this.isDragging = false;
        }
      }
    },
    drawImage(imageSource, squareX = null, squareY = null) {
      const vm = this;
      vm.image = new Image();
      vm.image.src = imageSource;
      vm.image.onload = function () {
        let canvas = vm.$refs.canvas;
        vm.offsetX = canvas.getBoundingClientRect().left;
        vm.offsetY = canvas.getBoundingClientRect().top;
        vm.canvasWidth = this.width;
        vm.canvasHeight = this.height;
        vm.ctx = canvas.getContext("2d");
        let wrh = this.width / this.height;
        vm.newWidth = canvas.width;
        vm.newHeight = vm.newWidth / wrh;
        if (vm.newHeight > canvas.height) {
          vm.newHeight = canvas.height;
          vm.newWidth = vm.newHeight * wrh;
        }
        canvas.width = vm.newWidth; // Set the canvas size to contain the dropped image while maintaining it's aspect ratio
        canvas.height = vm.newHeight;
        vm.ctx.drawImage(this, 0, 0, vm.newWidth, vm.newHeight);
        vm.$emit("dropped", canvas.toDataURL()); // Emit the 'dropped' event with the new image source
        vm.ctx.lineWidth = 8;
        vm.ctx.strokeStyle = "rgba(0, 217, 177)";
        vm.ctx.beginPath();
        if (!squareX || !squareY) {
          vm.startX = vm.newWidth / 2 - vm.newWidth / 8;
          vm.startY = vm.newHeight / 2 - vm.newWidth / 8;
        } else {
          vm.startX = squareX;
          vm.startY = squareY;
        }

        vm.ctx.rect(vm.startX, vm.startY, vm.newWidth / 4, vm.newWidth / 4);
        vm.ctx.stroke(); // Draws the rectangle

        vm.$emit("dragend", {
          startX: vm.startX,
          startY: vm.startY,
        }); // Emit the 'dragend' event when finished dragging the red rectangle which will be the area to crop
        // (startX, startY) will be the starting point to start cropping the image

        vm.isDragging = false;
      };
    },
    handleMouseDown(e) {
      // Handles the mousedown event for the red rectangle
      let mouseX = e.offsetX;
      let mouseY = e.offsetY;

      // Checks if the user is dragging inside the rectangle's area
      if (
        mouseX >= this.startX &&
        mouseX <= this.startX + this.newWidth / 4 &&
        mouseY >= this.startY &&
        mouseY <= this.startY + this.newWidth / 4 &&
        !this.isDragging
      ) {
        this.isDragging = true;
        this.canvasMouseX = mouseX;
        this.canvasMouseY = mouseY;
      }
    },
    handleMouseOut(e) {
      if (this.isDragging) {
        this.canvasMouseX = e.offsetX;
        this.canvasMouseY = e.offsetY;
        this.startX = this.canvasMouseX;
        this.startY = this.canvasMouseY;
        this.$emit("dragend", {
          startX: this.startX,
          startY: this.startY,
        }); // (startX, startY) will be the starting point to start cropping the image
      }
      this.isDragging = false;
    },
    handleMouseMove(e) {
      // Draws the rectangle in the corresponding dragged position
      const vm = this;
      if (this.isDragging) {
        this.canvasMouseX = e.offsetX;
        this.canvasMouseY = e.offsetY;
        this.startX = vm.canvasMouseX;
        this.startY = vm.canvasMouseY;

        vm.ctx.clearRect(0, 0, vm.newWidth, vm.newHeight);
        vm.ctx.drawImage(vm.image, 0, 0, vm.newWidth, vm.newHeight);
        vm.ctx.beginPath();
        vm.ctx.rect(vm.startX, vm.startY, vm.newWidth / 4, vm.newWidth / 4);
        vm.ctx.stroke();
      }
    },
  },
};
</script>



<style scoped>
/* TO DO after uploading coding challenge: replace for SCSS files, add a SASS loader, responsiveness :) */
h1 {
  width: 100%;
  text-align: center;
  color: rgb(33, 44, 49);
  font-weight: bold;
}

h1.invalid-file-text {
  margin: 0;
  position: absolute;
  top: calc(500px / 2 - 74px / 2);
}
h1.drop-image-text {
  margin: 0;
  position: absolute;
  top: calc(500px / 2 - 37px / 2);
}
.drop {
  width: 500px;
  height: 500px;
  background-color: #eee;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 1rem;
  transition: background-color 0.2s ease-in-out;
  border-radius: 4px;
  /* background-image: url("data:image/svg+xml,%3csvg width='100%25' height='100%25' xmlns='http://www.w3.org/2000/svg'%3e%3crect width='100%25' height='100%25' fill='none' rx='4' ry='4' stroke='%23333' stroke-width='6' stroke-dasharray='6%2c 14' stroke-dashoffset='12' stroke-linecap='square'/%3e%3c/svg%3e"); */
}

.drop:hover {
  background-color: #999;
}

.isDragging {
  background-color: #999;
}
img {
  width: 100%;
  height: 100%;
  object-fit: contain;
}

img.dropped-image {
  max-width: 500px;
  max-height: 500px;
}

.dropArea {
  position: relative;
}

.draggable {
  width: 100px;
  height: 100px;
  border: 2px solid white;
  position: absolute;
  top: 0;
}

.dragged {
  cursor: grabbing;
}
</style>