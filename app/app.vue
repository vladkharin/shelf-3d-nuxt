<template>
  <div class="container">
    <ClientOnly>
      <div class="scene-wrapper">
        <ShelfScene
          :key="`${config.width}-${config.height}`"
          :width="config.width"
          :height="config.height"
          :depth="config.depth"
          :rows="config.rows"
          :cols="config.cols"
          :color="config.color"
        />
      </div>
      <UIControllPanel @update:config="updateConfig" />
    </ClientOnly>
  </div>
</template>

<script setup lang="ts">
import ShelfScene from "./components/ShelfScene.vue";
import UIControllPanel from "./components/UIControllPanel.vue";
import { ref } from "vue";

defineOptions({
  ssr: false,
});

const config = ref({
  width: 1.075,
  height: 1.428,
  depth: 0.3,
  rows: 3,
  cols: 4,
  color: "#ffffff",
});

function updateConfig(newConfig: any) {
  Object.assign(config.value, newConfig);
}
</script>

<style scoped>
.container {
  display: flex;
  height: 100vh;
  gap: 20px;
  padding: 20px;
}

.scene-wrapper {
  flex: 1;
  background: #eee;
  border-radius: 8px;
  overflow: hidden;
}

@media (max-width: 768px) {
  .container {
    flex-direction: column;
  }
}
</style>
