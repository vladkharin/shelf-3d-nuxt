<template>
  <div class="controls-panel">
    <h3>Форма</h3>
    <div class="shape-buttons">
      <button :class="{ active: shape === 'infinite' }" @click="shape = 'infinite'">∞</button>
      <button :class="{ active: shape === 'grid' }" @click="shape = 'grid'">🞄</button>
      <button :class="{ active: shape === 'square' }" @click="shape = 'square'">□</button>
      <button :class="{ active: shape === 'image' }" @click="shape = 'image'">🖼️</button>
    </div>

    <div class="slider">
      <label>Ширина</label>
      <input type="range" v-model.number="width" min="0.5" max="2" step="0.001" />
      <span>{{ width.toFixed(3) }}</span>
    </div>

    <div class="slider">
      <label>Высота</label>
      <input type="range" v-model.number="height" min="0.5" max="3" step="0.001" />
      <span>{{ height.toFixed(3) }}</span>
    </div>

    <div class="slider">
      <label>Глубина</label>
      <input type="range" v-model.number="depth" min="0.1" max="1" step="0.001" />
      <span>{{ depth.toFixed(3) }}</span>
    </div>

    <div class="grid-controls">
      <label>Ячейки</label>
      <input type="number" v-model.number="rows" min="1" max="5" />
      <span>×</span>
      <input type="number" v-model.number="cols" min="1" max="5" />
    </div>

    <div class="color-selector">
      <label>Корпус</label>
      <div class="colors">
        <div
          v-for="color in colors"
          :key="color"
          :class="{ selected: selectedColor === color }"
          :style="{ backgroundColor: color }"
          @click="selectedColor = color"
        ></div>
      </div>
      <p>{{ colorLabels[selectedColor] || "Другой" }}</p>
    </div>

    <div class="buttons">
      <button>Сброс</button>
      <button>Случайно</button>
      <button>Рисуйте на модели</button>
    </div>

    <div class="help">
      <h4>Помощь</h4>
      <button>Чат</button>
      <button>Youtube</button>
      <button>VK-видео</button>
    </div>

    <button class="advanced">+ расширенные настройки</button>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from "vue";
defineOptions({ ssr: false });

const shape = ref("infinite");
const width = ref(1.075);
const height = ref(1.428);
const depth = ref(0.3);
const rows = ref(3);
const cols = ref(4);
const selectedColor = ref("#ffffff");

const colors = ["#ffffff", "#d2b48c", "#cccccc", "#8b4513", "#000000"];
const colorLabels: Record<string, string> = {
  "#ffffff": "Белый. ЛДСП 16мм",
  "#d2b48c": "Бежевый",
  "#cccccc": "Серый",
  "#8b4513": "Дерево",
  "#000000": "Черный",
};

const emit = defineEmits(["update:config"]);
const config = computed(() => ({
  width: width.value,
  height: height.value,
  depth: depth.value,
  rows: rows.value,
  cols: cols.value,
  color: selectedColor.value,
}));

watch(config, (newConfig) => {
  emit("update:config", newConfig);
});
</script>

<style scoped>
.controls-panel {
  padding: 20px;
  background: #fff;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  font-family: "Play", sans-serif;
}

.shape-buttons button {
  margin-right: 5px;
  padding: 5px 10px;
  border: 1px solid #ddd;
  background: #f5f5f5;
  cursor: pointer;
}

.shape-buttons button.active {
  background: #ff6b6b;
  color: white;
}

.slider {
  display: flex;
  align-items: center;
  margin: 15px 0;
}

.slider label {
  width: 80px;
}

.slider input[type="range"] {
  flex: 1;
  margin: 0 10px;
}

.slider span {
  width: 60px;
  text-align: center;
}

.grid-controls {
  display: flex;
  align-items: center;
  margin: 15px 0;
}

.grid-controls label {
  width: 80px;
}

.grid-controls input {
  width: 50px;
  margin: 0 5px;
}

.color-selector {
  margin: 15px 0;
}

.color-selector label {
  display: block;
  margin-bottom: 5px;
}

.colors {
  display: flex;
  gap: 5px;
  margin-bottom: 5px;
}

.colors div {
  width: 30px;
  height: 30px;
  border: 2px solid #ddd;
  cursor: pointer;
  border-radius: 50%;
}

.colors div.selected {
  border-color: #ff6b6b;
}

.buttons {
  display: flex;
  gap: 5px;
  margin: 15px 0;
}

.buttons button {
  padding: 5px 10px;
  border: 1px solid #ddd;
  background: #f5f5f5;
  cursor: pointer;
}

.help h4 {
  margin: 15px 0 5px;
}

.help button {
  margin-right: 5px;
  padding: 5px 10px;
  border: 1px solid #ddd;
  background: #f5f5f5;
  cursor: pointer;
}

.advanced {
  margin-top: 15px;
  padding: 5px 10px;
  border: 1px solid #ddd;
  background: #f5f5f5;
  cursor: pointer;
}
</style>
