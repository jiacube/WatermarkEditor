<template>
  <div class="watermark-editor">
    <div class="upload-section">
      <input type="file" @change="handleImageUpload" accept="image/*" />
    </div>

    <div v-if="imageSrc" class="editor-section">
      <div class="preview-container">
        <div class="image-wrapper" ref="imageWrapper">
          <img :src="imageSrc" ref="image" class="original-image" />
          <div
            class="watermark"
            :style="{
              top: watermarkPosition.y + 'px',
              left: watermarkPosition.x + 'px',
              fontSize: watermarkOptions.fontSize + 'px',
              color: watermarkOptions.color,
              opacity: watermarkOptions.opacity,
              transform: `rotate(${watermarkOptions.rotation}deg)`,
            }"
            @mousedown="startDrag"
          >
            {{ watermarkOptions.text }}
          </div>
        </div>
      </div>

      <div class="controls">
        <div class="watermark-controls">
          <h3>水印设置</h3>
          <div class="control-group">
            <label>水印文字:</label>
            <input v-model="watermarkOptions.text" type="text" />
          </div>
          <div class="control-group">
            <label>字体大小:</label>
            <input v-model.number="watermarkOptions.fontSize" type="range" min="10" max="72" />
            <span>{{ watermarkOptions.fontSize }}px</span>
          </div>
          <div class="control-group">
            <label>颜色:</label>
            <input v-model="watermarkOptions.color" type="color" />
          </div>
          <div class="control-group">
            <label>透明度:</label>
            <input v-model.number="watermarkOptions.opacity" type="range" min="0.1" max="1" step="0.1" />
            <span>{{ watermarkOptions.opacity }}</span>
          </div>
          <div class="control-group">
            <label>旋转角度:</label>
            <input v-model.number="watermarkOptions.rotation" type="range" min="0" max="360" />
            <span>{{ watermarkOptions.rotation }}°</span>
          </div>
        </div>

        <div class="action-buttons">
          <button @click="resetWatermark">重置位置</button>
          <button @click="downloadImage" class="download-btn">下载图片</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, reactive } from "vue";
import html2canvas from "html2canvas";

export default {
  setup() {
    const imageSrc = ref("");
    const imageWrapper = ref(null);
    const image = ref(null);

    const watermarkOptions = reactive({
      text: "我的水印",
      fontSize: 24,
      color: "#ffffff",
      opacity: 0.7,
      rotation: 0,
    });

    const watermarkPosition = reactive({
      x: 20,
      y: 20,
      isDragging: false,
    });

    const handleImageUpload = (e) => {
      const file = e.target.files[0];
      if (file) {
        const reader = new FileReader();
        reader.onload = (event) => {
          imageSrc.value = event.target.result;
        };
        reader.readAsDataURL(file);
      }
    };

    const startDrag = (e) => {
      watermarkPosition.isDragging = true;
      document.addEventListener("mousemove", dragWatermark);
      document.addEventListener("mouseup", stopDrag);
      e.preventDefault();
    };

    const dragWatermark = (e) => {
      if (!watermarkPosition.isDragging) return;

      const wrapperRect = imageWrapper.value.getBoundingClientRect();
      const x = e.clientX - wrapperRect.left;
      const y = e.clientY - wrapperRect.top;

      // 限制水印在图片范围内
      if (x >= 0 && x <= wrapperRect.width && y >= 0 && y <= wrapperRect.height) {
        watermarkPosition.x = x;
        watermarkPosition.y = y;
      }
    };

    const stopDrag = () => {
      watermarkPosition.isDragging = false;
      document.removeEventListener("mousemove", dragWatermark);
      document.removeEventListener("mouseup", stopDrag);
    };

    const resetWatermark = () => {
      watermarkPosition.x = 20;
      watermarkPosition.y = 20;
      watermarkOptions.rotation = 0;
    };

    const downloadImage = async () => {
      if (!imageSrc.value) return;

      try {
        const canvas = await html2canvas(imageWrapper.value, {
          backgroundColor: null,
          logging: false,
          useCORS: true,
        });

        const link = document.createElement("a");
        link.download = "watermarked-image.png";
        link.href = canvas.toDataURL("image/png");
        link.click();
      } catch (error) {
        console.error("Error generating image:", error);
      }
    };

    return {
      imageSrc,
      imageWrapper,
      image,
      watermarkOptions,
      watermarkPosition,
      handleImageUpload,
      startDrag,
      resetWatermark,
      downloadImage,
    };
  },
};
</script>

<style scoped>
.watermark-editor {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.upload-section {
  margin-bottom: 20px;
}

.editor-section {
  display: flex;
  gap: 20px;
}

.preview-container {
  flex: 1;
  border: 1px solid #ddd;
  padding: 10px;
  background-color: #f5f5f5;
}

.image-wrapper {
  position: relative;
  display: inline-block;
}

.original-image {
  max-width: 100%;
  max-height: 500px;
  display: block;
}

.watermark {
  position: absolute;
  cursor: move;
  user-select: none;
  text-shadow: 1px 1px 3px rgba(0, 0, 0, 0.5);
}

.controls {
  width: 300px;
  display: flex;
  flex-direction: column;
  gap: 20px;
}

/* 手机端响应式布局 */
@media (max-width: 768px) {
  .editor-section {
    flex-direction: column-reverse;
  }

  .controls {
    width: 100%;
  }

  .preview-container {
    margin-top: 20px;
  }

  .original-image {
    max-height: 300px;
  }
}

.watermark-controls {
  background: #f9f9f9;
  padding: 15px;
  border-radius: 5px;
}

.control-group {
  margin-bottom: 15px;
}

.control-group label {
  display: block;
  margin-bottom: 5px;
  font-weight: bold;
}

.control-group input[type="text"],
.control-group input[type="range"],
.control-group input[type="color"] {
  width: 100%;
}

.action-buttons {
  display: flex;
  gap: 10px;
}

button {
  padding: 8px 15px;
  background: #4caf50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

button:hover {
  background: #45a049;
}

.download-btn {
  background: #2196f3;
}

.download-btn:hover {
  background: #0b7dda;
}
</style>
