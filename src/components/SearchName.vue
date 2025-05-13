<template>
  <div class="search-name">
    <div class="d-flex align-items-center">
      <input
        type="text"
        class="form-control w-50 me-2"
        placeholder="請輸入營區名稱關鍵字"
        v-model="keyword"
        @keyup.enter="handleSearch"
      />
      <button
        class="btn btn-success"
        :disabled="loading"
        @click="handleSearch"
      >
        <span v-if="loading">搜尋中...</span>
        <span v-else>搜尋</span>
      </button>
    </div>
  </div>
</template>


<script setup>
import { ref } from "vue";

const keyword = ref("");
const loading = ref(false);
const emits = defineEmits(["search"]);

async function handleSearch() {
  // if (!keyword.value.trim()) return;
  loading.value = true;
  await emits("search", keyword.value);
  loading.value = false;
}
</script>

<style scoped>
.search-name {
  margin-bottom: 20px;
}
</style>
