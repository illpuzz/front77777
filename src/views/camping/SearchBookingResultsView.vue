<template>
  <div class="container">
    <h2 class="my-4">搜尋結果</h2>
    <BookingDateSummary />
    <div v-if="loading">載入中...</div>
    <div v-else-if="groupedResults.length === 0">查無符合條件的營區</div>
    <div v-else class="row">
      <!-- 左側：營位卡片 -->
      <div class="col-lg-8">
        <div
          v-for="group in groupedResults"
          :key="group.campAreaId"
          class="card mb-4 p-3 shadow-sm"
        >
          <div class="row g-3">
            <div class="col-md-4">
              <img
                :src="getImageUrl(group.campAreaImageUrl)"
                alt="營區圖片"
                class="img-fluid rounded"
                @error="handleImageError($event)"
              />
            </div>
            <div class="col-md-8">
              <h5>{{ group.campAreaName }}</h5>
              <p class="text-muted">地址：{{ group.location }}</p>
              <ul class="list-group list-group-flush">
                <li
                  v-for="spot in group.spots"
                  :key="spot.campSpotId"
                  class="list-group-item d-flex justify-content-between align-items-center"
                >
                  <div>
                    <strong>{{ spot.spotName }}</strong
                    >｜ 可訂帳數：{{ spot.availableTents }}｜ 平日：{{
                      spot.pricePerNight
                    }}｜ 假日：{{ spot.priceWeekend }}
                  </div>

                  <div class="d-flex align-items-center">
                    <label class="me-2">帳數：</label>
                    <select
                      :value="selectedSpots[spot.campSpotId] || 0"
                      @change="
                        (e) => updateQuantity(spot, Number(e.target.value))
                      "
                    >
                      <option value="0">0</option>
                      <option v-for="n in spot.maxTents" :key="n" :value="n">
                        {{ n }}
                      </option>
                    </select>
                  </div>
                </li>
              </ul>
            </div>
          </div>
        </div>
      </div>

      <!-- 右側：購物車 -->
      <div class="col-lg-4">
        <BookingCart />
      </div>
    </div>
  </div>
</template>

<script setup>
import BookingCart from "@/components/BookingCart.vue";
import BookingDateSummary from "@/components/BookingDateSummary.vue";
import { ref, onMounted, computed } from "vue";
import axios from "@/plugins/axios.js";
import { useCampSearchStore } from "@/stores/campSearchStore";
import { useBookingCartStore } from "@/stores/bookingCartStore";

const results = ref([]);
const loading = ref(true);

const searchStore = useCampSearchStore();
const payload = searchStore.searchPayload;

const cartStore = useBookingCartStore();
const selectedSpots = ref({}); // { [campSpotId]: quantity }

const checkInDate = computed(() => searchStore.searchPayload?.checkInDate);

const isWeekend = (dateStr) => {
  const date = new Date(dateStr);
  const day = date.getDay();
  return day === 0 || day === 6;
};

const updateQuantity = (spot, quantity) => {
  selectedSpots.value[spot.campSpotId] = quantity;

  const isHoliday = isWeekend(checkInDate.value);
  const unitPrice = isHoliday ? spot.priceWeekend : spot.pricePerNight;
  const subtotal = unitPrice * quantity ;
  const searchStore = useCampSearchStore();


  cartStore.updateSpotQuantity(spot.campSpotId, {
    spotId: spot.campSpotId,
    spotName: spot.spotName,
    quantity,
    unitPrice,
    subtotal,
  });
};

const getImageUrl = (url) => {
  return url ? `${import.meta.env.VITE_API_URL}${url}` : "/img/no-image.png";
};

const handleImageError = (event) => {
  event.target.src = "/img/no-image.png";
};

// 分組處理：將 spots 根據 campAreaId 整理成單一陣列
const groupedResults = computed(() => {
  const map = new Map();

  for (const item of results.value) {
    if (!map.has(item.campAreaId)) {
      map.set(item.campAreaId, {
        campAreaId: item.campAreaId,
        campAreaName: item.campAreaName,
        location: item.location,
        campAreaImageUrl: item.campAreaImageUrl,
        spots: [],
      });
    }
    map.get(item.campAreaId).spots.push(item);
  }

  return Array.from(map.values());
});

onMounted(async () => {
  if (!payload) {
    console.warn("⚠️ 沒有搜尋條件！");
    loading.value = false;
    return;
  }

  try {
    const res = await axios.post("/api/public/camp-spots/search", payload);
    results.value = res.data.list || [];
  } catch (err) {
    console.error("❌ 搜尋失敗", err);
  } finally {
    loading.value = false;
  }
});
</script>

<style>
.card {
  border-radius: 12px;
}

img.img-fluid {
  max-height: 200px;
  object-fit: cover;
}
</style>
