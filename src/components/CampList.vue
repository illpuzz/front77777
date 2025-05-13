<template>
  <div class="container">
    <div class="row gy-4">
      <div class="col-md-6" v-for="item in camps" :key="item.campAreaId">
        <CampCard :campArea="item" />
      </div>
    </div>

    <!-- 分頁按鈕 -->
    <div class="pagination mt-4" v-if="total > rows">
      <button class="page-btn" :disabled="currentPage === 1" @click="goToPage(currentPage - 1)">上一頁</button>
      <button
        class="page-btn"
        v-for="page in totalPages"
        :key="page"
        :class="{ active: page === currentPage }"
        @click="goToPage(page)"
      >
        {{ page }}
      </button>
      <button class="page-btn" :disabled="currentPage === totalPages" @click="goToPage(currentPage + 1)">下一頁</button>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import CampCard from './CampCard.vue';
import axiosapi from '@/plugins/axios.js';
import Swal from 'sweetalert2';

const camps = ref([]);
const total = ref(0);
const rows = ref(10);
const currentPage = ref(1);
const totalPages = ref(1);
const currentKeyword = ref('');
const currentFeatureIds = ref([]);

defineExpose({ callFind });

async function callFind(nameKeyword = '', featureIds = [], page = 1) {
  Swal.fire({ title: '載入中...', allowOutsideClick: false, didOpen: () => Swal.showLoading() });
  const body = {
    start: (page - 1) * rows.value,
    rows: rows.value,
    sort: 'campAreaId',
    dir: false,
    campAreaName: nameKeyword,
    featureIds: featureIds
  };

  try {
    console.log("🏁 callFind 呼叫進入"); // 放在 CampList.vue 的 callFind() 最上面
    const response = await axiosapi.post('/api/public/camp-areas/filter', body);
    console.log("呼叫 filter 方法");
    camps.value = response.data.list;
    total.value = response.data.count;
    totalPages.value = Math.ceil(total.value / rows.value);
    currentKeyword.value = nameKeyword;
    currentFeatureIds.value = featureIds;
  } catch (error) {
    Swal.fire({ icon: 'error', text: '訊息：' + error.message });
  } finally {
    Swal.close();
  }
}

function goToPage(page) {
  if (page < 1 || page > totalPages.value) return;
  currentPage.value = page;
  callFind(currentKeyword.value, currentFeatureIds.value, page);
}
</script>

<style scoped>
.page-btn {
  padding: 8px 12px;
  border: 1px solid #ccc;
  background: white;
  border-radius: 4px;
  margin: 0 2px;
  cursor: pointer;
}
.page-btn.active {
  background-color: #ff7f00;
  color: white;
  border-color: #ff7f00;
}
.page-btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}
</style>
