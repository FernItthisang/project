<template>
  <div id="app">
    <input type="file" @change="onFileChange" multiple />
    <button @click="exportLayout">Export Layout</button>
    <div class="canvas" ref="canvas">
      <div
        v-for="(image, index) in images"
        :key="index"
        :style="{ top: image.top + 'px', left: image.left + 'px', width: image.width + 'px', height: image.height + 'px' }"
        class="draggable"
        @mousedown="onDragStart($event, index)"
      >
        <img :src="image.url" :style="{ width: '100%', height: '100%' }" />
        <div class="resize-handle" @mousedown="onResizeStart($event, index, 'se')"></div>
      </div>
    </div>
  </div>
</template>

<script>
import html2canvas from 'html2canvas';

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
      this.draggingImageIndex = index;
      this.dragStartX = event.clientX - this.images[index].left;
      this.dragStartY = event.clientY - this.images[index].top;
      document.addEventListener('mousemove', this.onDrag);
      document.addEventListener('mouseup', this.onDragEnd);
    },
    onDrag(event) {
      if (this.draggingImageIndex !== null) {
        this.images[this.draggingImageIndex].top = event.clientY - this.dragStartY;
        this.images[this.draggingImageIndex].left = event.clientX - this.dragStartX;
      }
    },
    onDragEnd() {
      this.draggingImageIndex = null;
      document.removeEventListener('mousemove', this.onDrag);
      document.removeEventListener('mouseup', this.onDragEnd);
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
    exportLayout() {
      const canvas = this.$refs.canvas;
      html2canvas(canvas).then((canvas) => {
        const link = document.createElement('a');
        link.download = 'layout.png';
        link.href = canvas.toDataURL();
        link.click();
      });
    },
  },
};
</script>


<style>
.canvas {
  width: 800px;
  height: 600px;
  border: 1px solid #000;
  position: relative;
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
</style>
