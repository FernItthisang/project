<template>
  <div id="app">
    <input type="file" @change="onFileChange" multiple />
    <div class="canvas" ref="canvas">
      <div
        v-for="(image, index) in images"
        :key="index"
        :style="{ top: image.top + 'px', left: image.left + 'px', width: image.width + 'px', height: image.height + 'px' }"
        class="draggable"
        @mousedown="onDragStart($event, index)"
        @touchstart="onDragStart($event, index)"
      >
        <img :src="image.url" :style="{ width: '100%', height: '100%' }" />
        <button @click.stop="removeImage(index)">Remove</button>
        <div class="resize-handle" @mousedown="onResizeStart($event, index)"></div>
      </div>
    </div>
  </div>
</template>

<script>

export default {
  data() {
    return {
      images: [],
      draggingImageIndex: null,
      resizingImageIndex: null,
      dragStartX: 0,
      dragStartY: 0,
      originalWidth: 0,
      originalHeight: 0,
    };
  },
  methods: {
    onFileChange(event) {
      const files = event.target.files;
      for (let i = 0; i < files.length; i++) {
        const reader = new FileReader();
        reader.onload = (e) => {
          this.images.push({ url: e.target.result, top: 0, left: 0, width: 100, height: 100 });
        };
        reader.readAsDataURL(files[i]);
      }
    },
    onDragStart(event, index) {
      const touch = event.touches ? event.touches[0] : event;
      this.draggingImageIndex = index;
      this.dragStartX = touch.clientX - this.images[index].left;
      this.dragStartY = touch.clientY - this.images[index].top;
      document.addEventListener('mousemove', this.onDrag);
      document.addEventListener('touchmove', this.onDrag);
      document.addEventListener('mouseup', this.onDragEnd);
      document.addEventListener('touchend', this.onDragEnd);
    },
    onDrag(event) {
      const touch = event.touches ? event.touches[0] : event;
      if (this.draggingImageIndex !== null) {
        this.images[this.draggingImageIndex].top = touch.clientY - this.dragStartY;
        this.images[this.draggingImageIndex].left = touch.clientX - this.dragStartX;
      }
    },
    onDragEnd() {
      this.draggingImageIndex = null;
      document.removeEventListener('mousemove', this.onDrag);
      document.removeEventListener('touchmove', this.onDrag);
      document.removeEventListener('mouseup', this.onDragEnd);
      document.removeEventListener('touchend', this.onDragEnd);
    },
    onResizeStart(event, index) {
      event.stopPropagation();
      this.resizingImageIndex = index;
      this.resizeStartX = event.clientX;
      this.resizeStartY = event.clientY;
      this.originalWidth = this.images[index].width;
      this.originalHeight = this.images[index].height;
      document.addEventListener('mousemove', this.onResize);
      document.addEventListener('mouseup', this.onResizeEnd);
    },
    onResize(event) {
      if (this.resizingImageIndex !== null) {
        const dx = event.clientX - this.resizeStartX;
        const dy = event.clientY - this.resizeStartY;
        this.images[this.resizingImageIndex].width = this.originalWidth + dx;
        this.images[this.resizingImageIndex].height = this.originalHeight + dy;
      }
    },
    onResizeEnd() {
      this.resizingImageIndex = null;
      document.removeEventListener('mousemove', this.onResize);
      document.removeEventListener('mouseup', this.onResizeEnd);
    },
    removeImage(index) {
      this.images.splice(index, 1);
    },
  },
};
</script>

<style>
.canvas {
  width: 100%;
  height: 600px;
  border: 1px solid #000;
  position: relative;
  touch-action: none;
}
.draggable {
  position: absolute;
  cursor: grab;
}
.resize-handle {
  width: 10px;
  height: 10px;
  background: #000;
  position: absolute;
  bottom: 0;
  right: 0;
  cursor: se-resize;
}
button {
  position: absolute;
  top: 0;
  right: 0;
  background: red;
  color: white;
  border: none;
  cursor: pointer;
}
</style>
