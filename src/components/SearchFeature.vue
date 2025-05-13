<template>
    <div class="feature-filter">
      <button
        v-for="feature in features"
        :key="feature.tagId"
        class="feature-btn"
        :class="{ active: selectedIds.includes(feature.tagId) }"
        @click="toggleFeature(feature.tagId)"
      >
        {{ feature.tagName }}
      </button>
    </div>
  </template>
  
  <script setup>
  import { ref } from 'vue';
  
  const props = defineProps({
    features: Array
  });
  const emits = defineEmits(['update:selectedFeatures']);
  const selectedIds = ref([]);
  
  function toggleFeature(id) {
    const index = selectedIds.value.indexOf(id);
    if (index > -1) {
      selectedIds.value.splice(index, 1);
    } else {
      selectedIds.value.push(id);
    }
    emits('update:selectedFeatures', selectedIds.value);
  }
  </script>
  
  <style scoped>
  .feature-filter {
    display: flex;
    flex-wrap: wrap;
    gap: 10px;
    margin-bottom: 20px;
  }
  
  /* 預設樣式 */
  .feature-btn {
    padding: 8px 14px;
    font-size: 16px;
    border: 1px solid #ccc;
    border-radius: 30px;
    background-color: white;
    color: #555;
    cursor: pointer;
    transition: all 0.3s ease;
  }
  
  /* 滑鼠 hover */
  .feature-btn:hover {
    border-color: #28a745;
    background-color: rgba(40, 167, 69, 0.05);
    color: #28a745;
  }
  
  /* 被選中的狀態 */
  .feature-btn.active {
    background-color: #28a745;
    color: white;
    border-color: #28a745;
    box-shadow: 0 0 8px rgba(40, 167, 69, 0.6);
  }
  </style>
  