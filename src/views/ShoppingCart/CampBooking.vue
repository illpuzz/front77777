
<template>
    <h1>謙哥營區購物車</h1>
  <div class="cart-container" v-if="isDataLoaded">
    <div class="cart-header">🛒 營地訂單資訊</div>

    <div v-for="item in campData" :key="item.spotId" class="cart-item">
      <h3>{{ item.campAreaName }}</h3>
      <div class="camp-image">
        <img :src="item.imageUrl" alt="營區圖片" />
      </div>
      <p>日期：{{ formattedStartDate }} - {{ formattedEndDate }}（共 {{ days }} 天）</p>
      <p>營位：{{ item.campSitesName }}</p>
      <p>營地點：{{ item.campSpotsName }}</p>
      <div class="item-actions">
        <label>帳篷數量：
          <input type="number" v-model.number="item.quantity" min="1" disabled />
        </label>
      </div>
      <p>每個帳篷/晚：${{ item.unitPrice }}</p>
    </div>

    <div class="cart-footer">
      <div class="cart-total">
        <span>總金額</span>
        <span>NT${{ totalPrice }}</span>
      </div>
      <button class="checkout-btn" @click="checkout">前往結帳</button>
    </div>
  </div>
</template>
<script setup>
import { onMounted, computed, ref } from 'vue';
import { useBookingCartStore } from '@/stores/bookingCartStore';
import { useCampSearchStore } from '@/stores/campSearchStore';
import axiosapi from '@/plugins/axios.js';
import { format, differenceInCalendarDays } from 'date-fns';

const path = import.meta.env.VITE_API_URL;
const cartStore = useBookingCartStore();
const searchStore = useCampSearchStore();
const cartItems = cartStore.selectedSpots;

const startDate = computed(() => searchStore.searchPayload?.checkInDate || '');
const endDate = computed(() => searchStore.searchPayload?.checkOutDate || '');

const isDataLoaded = ref(false);
const campData = ref([]);

const formattedStartDate = computed(() => format(new Date(startDate.value), 'yyyy/MM/dd'));
const formattedEndDate = computed(() => format(new Date(endDate.value), 'yyyy/MM/dd'));

const days = computed(() => {
  const start = new Date(startDate.value);
  const end = new Date(endDate.value);
  return differenceInCalendarDays(end, start) || 1;
});

const totalPrice = computed(() => {
  return cartItems.reduce((sum, item) => sum + item.unitPrice * item.quantity * days.value, 0);
});

async function fetchCampDetails() {
  const promises = cartItems.map(async (item) => {
    const { data } = await axiosapi.get(`/api/public/camp-spot/${item.spotId}`);
    return {
      ...item,
      campAreaName: data.campAreaName,
      campSitesName: data.campSiteName,
      campSpotsName: data.spotName,
      imageUrl: `${path}${data.imageUrl}`,
    };
  });
  campData.value = await Promise.all(promises);
  isDataLoaded.value = true;
}

function checkout() {
  alert(`結帳金額：${totalPrice.value}`);
}

onMounted(fetchCampDetails);
</script>

<style scoped>
/* 保留原有樣式 */
.cart-container {
  max-width: 500px;
  margin: 2rem auto;
  background-color: white;
  border-radius: 16px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  overflow: hidden;
}

.cart-header {
  background-color: #A8D5BA;
  padding: 1rem;
  font-size: 1.25rem;
  font-weight: bold;
  text-align: center;
  color: #4A7C59;
}

.cart-item {
  padding: 1rem;
  border-bottom: 1px solid #dde9da;
}

.cart-item h3 {
  margin: 0 0 0.25rem 0;
  color: #4A7C59;
}

.cart-item p {
  margin: 0.25rem 0;
  color: #444;
  font-size: 0.95rem;
}

.camp-image img {
  width: 100%;
  height: auto;
  border-radius: 8px;
}

.item-actions {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-top: 0.5rem;
}

.item-actions input {
  width: 40px;
  text-align: center;
}

.cart-footer {
  padding: 1rem;
  background-color: #f0f5f2;
}

.cart-total {
  display: flex;
  justify-content: space-between;
  font-weight: bold;
  color: #4A7C59;
  margin-bottom: 1rem;
}

.checkout-btn {
  width: 100%;
  padding: 0.75rem;
  background-color: #A8D5BA;
  border: none;
  color: white;
  font-size: 1rem;
  font-weight: bold;
  border-radius: 8px;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

.checkout-btn:hover {
  background-color: #4A7C59;
}
</style>
