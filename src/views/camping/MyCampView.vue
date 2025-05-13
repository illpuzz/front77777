<template>
  <div class="container my-5">
    <h2>我的營地</h2>

    <!-- ⏳ 載入中 -->
    <div v-if="isLoading" class="text-center my-5">
      <div class="spinner-border text-primary" role="status">
        <span class="visually-hidden">Loading...</span>
      </div>
      <p class="mt-2 text-muted">載入中...</p>
    </div>

    <!-- ✅ 無營地 -->
    <div v-else-if="campAreas.length === 0" class="text-center my-5">
      <p class="text-muted">您尚未建立任何營地</p>
      <router-link to="/my-camp/create" class="btn btn-success">
        ➕ 新增營區
      </router-link>
    </div>

    <!-- ✅ 有營地 -->
    <div v-else>
      <router-link to="/my-camp/create" class="btn btn-success mb-3">
        ➕ 新增營地
      </router-link>

      <div
        v-for="camp in campAreas"
        :key="camp.campAreaId"
        class="card mb-4 shadow-sm"
      >
        <div class="card-body">
          <!-- 營區標題 -->
          <div class="mb-3 d-flex flex-wrap align-items-center">
            <div class="me-auto d-flex align-items-center gap-2">
              <button
                class="btn btn-sm btn-outline-secondary"
                @click="toggleExpand(camp)"
              >
                <span v-if="camp.isExpanded">▼</span>
                <span v-else>▶</span>
              </button>
              <div>
                <h5 class="mb-0">{{ camp.campAreaName }}</h5>
                <small class="text-muted">
                  {{ camp.location }}｜{{ camp.address }}
                </small>
              </div>
            </div>

            <div class="d-flex flex-wrap gap-2 ms-auto">
              <router-link
                :to="`/my-camp/edit/${camp.campAreaId}`"
                class="btn btn-primary btn-sm"
              >
                編輯
              </router-link>
              <button
                class="btn btn-danger btn-sm"
                @click="deleteCampArea(camp.campAreaId)"
              >
                刪除
              </button>
              <select
                v-model="camp.status"
                @change="updateStatus(camp)"
                class="form-select form-select-sm w-auto"
              >
                <option value="Active">營運中</option>
                <option value="Inactive">已關閉</option>
              </select>
            </div>
          </div>

          <!-- 展開內容 -->
          <div v-show="camp.isExpanded">
            <div class="row">
              <div
                v-for="site in camp.campSites"
                :key="site.campSiteId"
                class="col-md-4 mb-3"
              >
                <div class="border rounded bg-light p-3 h-100">
                  <h6 class="mb-3">
                    🏕️ {{ site.campSiteName }}（{{ site.siteType }}）
                  </h6>

                  <div
                    v-for="spot in site.campSpots"
                    :key="spot.campSpotId"
                    class="border-start ps-3 mb-3"
                  >
                    <p class="mb-1">
                      <strong>名稱：</strong>{{ spot.spotName }}
                    </p>
                    <p class="mb-1">
                      <strong>大小：</strong>{{ spot.areaSize }} ｜ 帳數：
                      {{ spot.maxTents }}
                    </p>
                    <p class="mb-1">
                      <strong>平日：</strong>${{ spot.pricePerNight }} ｜
                      <strong>假日：</strong>${{ spot.priceHoliday }}
                    </p>
                    <div class="d-flex flex-wrap gap-2">
                      <router-link
                        :to="`/my-camp/edit-site/${spot.campSpotId}`"
                        class="btn btn-outline-primary btn-sm"
                      >
                        編輯營位
                      </router-link>
                      <button
                        class="btn btn-outline-danger btn-sm"
                        @click="deleteCampSpot(spot.campSpotId)"
                      >
                        刪除營位
                      </button>
                    </div>
                  </div>
                </div>
              </div>
            </div>

            <!-- ➕ 新增營地營位按鈕 -->
            <div class="mt-3">
              <router-link
                :to="`/my-camp/create-site/${camp.campAreaId}`"
                class="btn btn-outline-success btn-sm"
              >
                ➕ 新增營地營位
              </router-link>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import axiosapi from "@/plugins/axios";

const campAreas = ref([]);
const isLoading = ref(true);

const fetchCampAreas = async () => {
  try {
    const res = await axiosapi.get("/api/owner/camp-areas/own");

    campAreas.value = res.data.list.map((camp) => {
      // 先排序每個 site 裡的 spots（可留著）
      const sortedSites = (camp.campSites || [])
        .slice()
        .sort((a, b) => (a.campSiteId || 0) - (b.campSiteId || 0))
        .map((site) => {
          const sortedSpots = (site.campSpots || []).slice().sort((a, b) => {
            return (a.campSpotId || 0) - (b.campSpotId || 0);
          });
          return {
            ...site,
            campSpots: sortedSpots,
          };
        });

      return {
        ...camp,
        isExpanded: false,
        campSites: sortedSites,
      };
    });
  } catch (error) {
    console.error("載入營地失敗", error);
  } finally {
    isLoading.value = false;
  }
};

const deleteCampSpot = async (campSpotId) => {
  if (!confirm("確定要刪除這個營位嗎？")) return;

  try {
    await axiosapi.delete(`/api/owner/camp-spots/${campSpotId}`);
    alert("營位刪除成功（");
    await fetchCampAreas(); // 重新載入營區
  } catch (error) {
    alert("刪除失敗，請稍後再試");
    console.error(error);
  }
};

const toggleExpand = (camp) => {
  camp.isExpanded = !camp.isExpanded;
};

const updateStatus = async (camp) => {
  try {
    await axiosapi.put(`/api/owner/camp-areas/${camp.campAreaId}/status`, {
      status: camp.status,
    });
    alert("營區狀態已更新");
    await fetchCampAreas();
  } catch (error) {
    alert("更新失敗，請稍後再試");
    console.error(error);
  }
};

const deleteCampArea = async (id) => {
  if (!confirm("確定要刪除這個營地嗎？此操作無法復原！")) return;

  try {
    await axiosapi.delete(`/api/owner/camp-areas/${id}`);
    alert("已成功刪除營地");
    await fetchCampAreas();
  } catch (error) {
    alert("刪除失敗，請稍後再試");
    console.error(error);
  }
};

onMounted(fetchCampAreas);
</script>
