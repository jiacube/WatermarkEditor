<template>
  <div class="app-container">
    <h1>照片水印添加工具</h1>
    <WatermarkEditor />
  </div>
</template>

<script>
import WatermarkEditor from "./components/WatermarkEditor.vue";

export default {
  components: {
    WatermarkEditor,
  },
};
</script>

<style>
.app-container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  font-family: Arial, sans-serif;
}
</style>
