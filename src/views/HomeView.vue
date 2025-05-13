<template>
  <div class="container">
    <!-- <button @click="goToOrders">訂位按鈕test</button> -->
    <BannerComponent />
    <SearchFilters />
    <SearchName @search="handleSearch" />
    <SearchFeature :features="allFeatures" @update:selectedFeatures="handleFeatureSelect" />
    <CampList ref="campListRef" />
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import SearchFilters from '@/components/SearchFilters.vue';
import CampList from '@/components/CampList.vue';
import SearchName from '@/components/SearchName.vue';
import SearchFeature from '@/components/SearchFeature.vue';
import axiosapi from "@/plugins/axios.js";
import BannerComponent from '@/components/BannerComponent.vue';

// // 訂位---------
// import { useRouter } from 'vue-router';
// import { useBookingStore } from '@/stores/useBookingStore';

// const bookingStore = useBookingStore()
// const router = useRouter();

// function goToOrders() {

//   const campSpotId = 1;
//   const startDate = new Date('2025-06-01');
//   const endDate = new Date('2025-06-03');

//   console.log("✅ 資料已寫入 Pinia", bookingStore.campSpotId, bookingStore.dateRange);
//   bookingStore.setBookingInfo(campSpotId, startDate, endDate)
//   router.push('/campBooking');
// }
// //-------------------


const campListRef = ref(null);
const allFeatures = ref([]);
const selectedFeatureIds = ref([]); 
const keyword = ref('');

async function fetchFeatures() {
const response = await axiosapi.get('/api/public/camp-areas/features');
allFeatures.value = response.data; 
console.log("呼叫feature方法" , allFeatures.value);
}

function handleSearch(kw) {
keyword.value = kw;
searchCamps();
}

function handleFeatureSelect(ids) {
selectedFeatureIds.value = ids;
console.log("selectedfeature: ",selectedFeatureIds.value);
searchCamps();
}

function searchCamps() {
if (campListRef.value) {
  campListRef.value.callFind(keyword.value, selectedFeatureIds.value);
}
}



onMounted(async () => {
  await fetchFeatures();  // 等特徵標籤載入完再查營地
  searchCamps();
});


</script>

<style scoped>
.page-title {
  font-size: 24px;
  margin-bottom: 20px;
  color: #333;
}
</style>