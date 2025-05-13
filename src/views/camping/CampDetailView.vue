<template>
  <div class="container my-5" v-if="campArea">
    <!-- 1. 營區名稱 -->
    <div class="text-center mb-4">
      <h1 class="display-5">{{ campArea.campAreaName }}</h1>
      <p class="text-muted">{{ campArea.location }}</p>
    </div>

    <!-- 2. 營區基本資訊 -->
    <div class="row mb-4">
      <div class="col-md-6">
        <ul class="list-group">
          <li class="list-group-item">
            <strong>地址：</strong>{{ campArea.address }}
          </li>
          <li class="list-group-item">
            <strong>電話：</strong>{{ campArea.phone }}
          </li>
          <li class="list-group-item">
            <strong>Email：</strong>{{ campArea.email }}
          </li>
          <li class="list-group-item">
            <strong>網站：</strong>
            <a :href="campArea.website_url" target="_blank">{{
              campArea.websiteUrl 
            }}</a>
          </li>
        </ul>
      </div>
      <div class="col-md-6">
        <ul class="list-group">
          <li class="list-group-item">
            <strong>海拔：</strong>{{ campArea.altitude }} 公尺
          </li>
          <li class="list-group-item">
            <strong>交通方式：</strong>{{ campArea.transportation }}
          </li>
          <li class="list-group-item">
            <strong>注意事項：</strong>{{ campArea.notice }}
          </li>
        </ul>
      </div>
    </div>

    <!-- 3. 營區圖片區 -->
    <div class="mb-5">
      <h2 class="mb-3">營區圖片</h2>
      <div class="row g-3">
        <div
          class="col-6 col-md-4"
          v-for="(img, index) in campArea.mainImageUrl"
          :key="index"
        >
          <img
            :src="path + img.imageUrl"
            class="img-fluid rounded shadow-sm camp-image"
            alt="營區圖片"
          />
        </div>
      </div>
    </div>

    <!-- 營位資訊表格 -->
    <div
      class="mb-5"
      v-if="campArea.campSites && campArea.campSites.length > 0"
    >
      <h2 class="mb-3">營位資訊</h2>
      <table class="table table-hover">
        <thead>
          <tr>
            <th>營位名稱</th>
            <th>型態</th>
            <th>尺寸</th>
            <th>帳數</th>
            <th>平日</th>
            <th>假日</th>
            <th>詳細</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="site in campArea.campSites" :key="site.campSiteId">
            <td>
              <!-- 傳到modal裡的已經是spot了取值就 spot.xxx -->
              <button
                class="btn btn-link p-0"
                @click="openSpotModal(site.campSpots?.[0])" 
                >
                {{ site.campSiteName }}
              </button>
            </td>
            <td>{{ site.siteType }}</td>
            <td>{{ site.campSpots?.[0]?.areaSize ?? "-" }}</td>
            <td>{{ site.campSpots?.[0]?.maxTents ?? "-" }}</td>
            <td>${{ site.campSpots?.[0]?.pricePerNight ?? "-" }}</td>
            <td>${{ site.campSpots?.[0]?.priceHoliday ?? "-" }}</td>
            <td>詳細按鈕</td>
          </tr>
        </tbody>
      </table>
    </div>
    <CampSpotModal v-model:spot="selectedSpot" />
    <!-- 5. 設施區 -->
    <div class="mb-5">
      <h2 class="mb-3">設施資訊</h2>

      <div class="row">
        <div class="col-md-4">
          <h5>設備</h5>
          <ul class="list-unstyled">
            <li
              v-for="(facility, index) in campArea.facilityNames"
              :key="index"
            >
              - {{ facility }}
            </li>
          </ul>
        </div>
        <div class="col-md-4">
          <h5>Wi-Fi</h5>
          <ul class="list-unstyled">
            <li v-for="(wifi, index) in campArea.wifiNames" :key="index">
              - {{ wifi }}
            </li>
          </ul>
        </div>
        <div class="col-md-4">
          <h5>浴室</h5>
          <ul class="list-unstyled">
            <li
              v-for="(bathroom, index) in campArea.bathroomNames"
              :key="index"
            >
              - {{ bathroom }}
            </li>
          </ul>
        </div>
      </div>

      <div class="row mt-4">
        <div class="col-md-6">
          <h5>停車場</h5>
          <ul class="list-unstyled">
            <li v-for="(parking, index) in campArea.parkingNames" :key="index">
              - {{ parking }}
            </li>
          </ul>
        </div>
        <div class="col-md-6">
          <h5>寵物政策</h5>
          <ul class="list-unstyled">
            <li v-for="(pet, index) in campArea.petPolicyNames" :key="index">
              - {{ pet }}
            </li>
          </ul>
        </div>
      </div>
    </div>
  </div>

  <!-- 載入中的 spinner -->
  <div v-else class="text-center my-5">
    <div class="spinner-border text-primary" role="status">
      <span class="visually-hidden">載入中...</span>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import { useRoute } from "vue-router";
import axiosapi from "@/plugins/axios.js";
import CampSpotModal from "@/components/CampSpotModal.vue";

const campArea = ref(null);
const path = import.meta.env.VITE_API_URL || ""; // 圖片網址前綴
const route = useRoute();
const selectedSpot = ref(null);

function openSpotModal(spot) {
  selectedSpot.value = spot;
}

onMounted(async () => {
  const id = route.params.id;
  const response = await axiosapi.get(
    `/api/public/camp-areas/${id}`
  );
  campArea.value = response.data.list[0];
  console.log(campArea.value);
});

function formatDate(dateString) {
  if (!dateString) return "";
  return new Date(dateString).toLocaleDateString();
}
</script>

<style scoped>
.camp-image {
  width: 350px;
  height: 230px;
  object-fit: cover; /* 重點：保持比例，但會裁切 */
  object-position: center; /* 圖片以中心點對齊裁切 */
}
</style>
