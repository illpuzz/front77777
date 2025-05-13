<template>
    <div class="container my-5">
      <h2>新增營地</h2>
      <form @submit.prevent="submitForm">
        <div class="row">
          <div class="mb-3 col-md-6">
            <label class="form-label">營地名稱</label>
            <input v-model="form.campAreaName" class="form-control" />
          </div>
  
          <div class="mb-3 col-md-6">
            <label class="form-label">地點</label>
            <input v-model="form.location" class="form-control" />
          </div>
  
          <div class="mb-3 col-md-6">
            <label class="form-label">地址</label>
            <input v-model="form.address" class="form-control" />
          </div>
  
          <div class="mb-3 col-md-6">
            <label class="form-label">電話</label>
            <input v-model="form.phone" class="form-control" />
          </div>
  
          <div class="mb-3 col-md-6">
            <label class="form-label">Email</label>
            <input v-model="form.email" class="form-control" />
          </div>
  
          <div class="mb-3 col-md-6">
            <label class="form-label">交通資訊</label>
            <textarea v-model="form.transportation" class="form-control" rows="2"></textarea>
          </div>
  
          <div class="mb-3 col-md-6">
            <label class="form-label">營區描述</label>
            <textarea v-model="form.description" class="form-control" rows="3"></textarea>
          </div>
  
          <div class="mb-3 col-md-6">
            <label class="form-label">座標（POINT 格式）</label>
            <input v-model="form.coordinates" class="form-control" placeholder="POINT (經度 緯度)" />
          </div>
  
          <div class="mb-3 col-md-6">
            <label class="form-label">網站網址</label>
            <input v-model="form.websiteUrl" class="form-control" />
          </div>
  
          <div class="mb-3 col-md-6">
            <label class="form-label">注意事項</label>
            <textarea v-model="form.notice" class="form-control" rows="2"></textarea>
          </div>
  
          <div class="mb-3 col-md-6">
            <label class="form-label">海拔高度 (m)</label>
            <input type="number" v-model="form.altitude" class="form-control" />
          </div>
        </div>
  
        <div class="mb-3">
          <label class="form-label">營區圖片</label>
          <input type="file" multiple @change="handleFileUpload" class="form-control" />
        </div>
  
        <div class="mb-3">
          <label class="form-label">特色</label>
          <div class="d-flex flex-wrap gap-3">
            <div v-for="tag in allFeatures" :key="tag.id" class="form-check" :style="{ minWidth: '150px' }">
              <input class="form-check-input" type="checkbox" :value="tag.tagId" v-model="form.featureTagIds" :id="`feature-${tag.id}`" />
              <label class="form-check-label" :for="`feature-${tag.id}`">{{ tag.tagName }}</label>
            </div>
          </div>
        </div>
  
        <div class="mb-3">
          <label class="form-label">WiFi 設施</label>
          <div class="d-flex flex-wrap gap-3">
            <div class="form-check" v-for="item in allWifis" :key="item.id">
              <input class="form-check-input" type="checkbox" :value="item.id" v-model="form.wifiIds" />
              <label class="form-check-label">{{ item.name }}</label>
            </div>
          </div>
        </div>
  
        <div class="mb-3">
          <label class="form-label">停車場</label>
          <div class="d-flex flex-wrap gap-3">
            <div class="form-check" v-for="item in allParkings" :key="item.id">
              <input class="form-check-input" type="checkbox" :value="item.id" v-model="form.parkingIds" />
              <label class="form-check-label">{{ item.name }}</label>
            </div>
          </div>
        </div>
  
        <div class="mb-3">
          <label class="form-label">設施</label>
          <div class="d-flex flex-wrap gap-3">
            <div class="form-check" v-for="item in allFacilities" :key="item.id">
              <input class="form-check-input" type="checkbox" :value="item.id" v-model="form.facilityIds" />
              <label class="form-check-label">{{ item.name }}</label>
            </div>
          </div>
        </div>
  
        <div class="mb-3">
          <label class="form-label">浴室</label>
          <div class="d-flex flex-wrap gap-3">
            <div class="form-check" v-for="item in allBathrooms" :key="item.id">
              <input class="form-check-input" type="checkbox" :value="item.id" v-model="form.bathroomIds" />
              <label class="form-check-label">{{ item.name }}</label>
            </div>
          </div>
        </div>
  
        <div class="mb-3">
          <label class="form-label">寵物政策</label>
          <div class="d-flex flex-wrap gap-3">
            <div class="form-check" v-for="item in allPetPolicies" :key="item.id">
              <input class="form-check-input" type="checkbox" :value="item.id" v-model="form.petPolicyIds" />
              <label class="form-check-label">{{ item.name }}</label>
            </div>
          </div>
        </div>
  
        <div class="mb-3">
          <label class="form-label">營地狀態</label>
          <select v-model="form.status" class="form-select">
            <option value="Active">營運中</option>
            <option value="Inactive">已關閉</option>
          </select>
        </div>
  
        <button type="submit" class="btn btn-success">儲存新增</button>
      </form>
    </div>
  </template>
  
  <script setup>
  import { ref, onMounted } from "vue";
  import { useRouter } from "vue-router";
  import axiosapi from "@/plugins/axios";
  
  const router = useRouter();
  
  const form = ref({
    campAreaName: "",
    location: "",
    address: "",
    phone: "",
    email: "",
    transportation: "",
    description: "",
    coordinates: "",
    websiteUrl: "",
    notice: "",
    altitude: null,
    featureTagIds: [],
    wifiIds: [],
    parkingIds: [],
    facilityIds: [],
    bathroomIds: [],
    petPolicyIds: [],
    status: "Active",
  });
  
  const imageFiles = ref([]);
  const allFeatures = ref([]);
  const allWifis = ref([]);
  const allParkings = ref([]);
  const allFacilities = ref([]);
  const allBathrooms = ref([]);
  const allPetPolicies = ref([]);
  
  const handleFileUpload = (event) => {
    const files = Array.from(event.target.files);
    imageFiles.value.push(...files);
  };
  
  const submitForm = async () => {
    const formData = new FormData();
    formData.append(
      "request",
      new Blob([JSON.stringify(form.value)], { type: "application/json" })
    );
    imageFiles.value.forEach((file) => {
      formData.append("images", file);
    });
  
    try {
      await axiosapi.post("/api/owner/camp-areas", formData, {
        headers: { "Content-Type": "multipart/form-data" },
      });
      alert("新增成功！");
      router.push("/my-camp");
    } catch (err) {
      console.error(err);
      alert("新增失敗，請稍後再試");
    }
  };
  
  onMounted(async () => {
    allFeatures.value = (await axiosapi.get("/api/public/camp-areas/features")).data;
    allWifis.value = (await axiosapi.get("/api/public/camp-areas/options/wifi")).data;
    allParkings.value = (await axiosapi.get("/api/public/camp-areas/options/parking")).data;
    allFacilities.value = (await axiosapi.get("/api/public/camp-areas/options/facility")).data;
    allBathrooms.value = (await axiosapi.get("/api/public/camp-areas/options/bathroom")).data;
    allPetPolicies.value = (await axiosapi.get("/api/public/camp-areas/options/pet-policy")).data;
  });
  </script>