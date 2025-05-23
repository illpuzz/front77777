<!-- src/views/reviews/ReviewsView.vue -->
<template>
    <div class="reviews-view">
      <!-- 管理員檢舉管理面板（僅管理員可見） -->
      <div v-if="isAdmin || isSuperAdmin" class="admin-controls mb-4">
        <div class="d-flex justify-content-between align-items-center">
          <h2 class="admin-title">
            <i class="bi bi-shield-check me-2"></i>管理員專區
            <span v-if="isSuperAdmin" class="badge bg-danger ms-2">超級管理員</span>
          </h2>
          <div class="control-buttons">
            <button 
              class="btn btn-admin-reviews me-2" 
              @click="adminView = 'reviews'"
              :class="{ active: adminView === 'reviews' }"
            >
              <i class="bi bi-list-stars me-1"></i>
              評價管理
            </button>
            <button 
              class="btn btn-admin-reports" 
              @click="adminView = 'reports'"
              :class="{ active: adminView === 'reports' }"
            >
              <i class="bi bi-flag-fill me-1"></i>
              檢舉管理
            </button>
          </div>
        </div>
      </div>
      
      <!-- 管理員檢舉管理面板 -->
      <AdminReportManager 
        v-if="(isAdmin || isSuperAdmin) && adminView === 'reports'"
        :currentUser="currentUserObject"
        @handle-report="handleReport"
      />
      
      <!-- 評價相關功能（所有用戶可見） -->
      <div v-if="adminView === 'reviews' || !(isAdmin || isSuperAdmin)">
        <ReviewFilter 
          @filter-applied="applyFilters"
          @add-review="handleAddReviewClick"
          :canAddReview="isLoggedIn && isRegularUser && !isCampOwner"
        />
        
        <!-- 使用路由參數的campSiteId -->
        <ReviewsList 
          :campSiteId="campSiteId"
          :filters="filters"
          :currentUser="currentUserObject"
          @handle-report="handleReport"
          @total-updated="updateTotalReviews"
          @toggle-like="handleToggleLike"
          ref="reviewsListRef"
        />
      </div>
      
      <!-- 新增評價 Modal -->
      <div class="modal fade" id="addReviewModal" tabindex="-1" aria-hidden="true">
        <div class="modal-dialog modal-lg">
          <div class="modal-content">
            <div class="modal-header">
              <h5 class="modal-title">新增評價</h5>
              <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
            </div>
            <div class="modal-body">
              <!-- 使用路由參數的campSiteId -->
              <AddReviewForm 
                :campSiteId="campSiteId"
                :userId="userId"
                @review-added="handleReviewAdded"
              />
            </div>
          </div>
        </div>
      </div>
      
      <!-- 管理員處理檢舉 Modal -->
      <div class="modal fade" id="handleReportModal" tabindex="-1" aria-hidden="true">
        <div class="modal-dialog">
          <div class="modal-content">
            <div class="modal-header">
              <h5 class="modal-title">處理檢舉</h5>
              <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
            </div>
            <div class="modal-body">
              <div v-if="selectedReport">
                <div class="alert alert-info">
                  <strong>檢舉類型：</strong> {{ selectedReport.target === 'review' ? '評價' : '回覆' }}
                  <hr>
                  <strong>檢舉原因：</strong> {{ getReportTypeName(selectedReport.reportType) }}
                  <hr>
                  <strong>詳細說明：</strong> {{ selectedReport.reason }}
                </div>
                
                <div class="form-group mb-3">
                  <label for="handlerNote" class="form-label">處理備註：</label>
                  <textarea id="handlerNote" class="form-control" v-model="reportHandlerNote" rows="3"></textarea>
                </div>
                
                <div class="d-flex gap-2 justify-content-end">
                  <button class="btn btn-outline-secondary" data-bs-dismiss="modal">取消</button>
                  <button class="btn btn-danger" @click="approveReport">批准檢舉並移除內容</button>
                  <button class="btn btn-success" @click="rejectReport">駁回檢舉</button>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
</template>
  
<script>
import { ref, reactive, onMounted, watch, computed } from 'vue';
import { Modal } from 'bootstrap';
import { useRoute } from 'vue-router';
import { useAuthStore } from '@/stores/auth';
import axiosapi from '@/plugins/axios.js'; // 使用配置好的axios實例

// 確保加載了Bootstrap樣式和腳本
import 'bootstrap/dist/css/bootstrap.min.css';
import 'bootstrap/dist/js/bootstrap.bundle.min.js';
  
// 引入組件
import AdminReportManager from '@/components/reviews/AdminReportManager.vue';
import ReviewFilter from '@/components/reviews/ReviewFilter.vue';
import ReviewsList from '@/components/reviews/ReviewsList.vue';
import AddReviewForm from '@/components/reviews/forms/AddReviewForm.vue';
  
export default {
  name: 'ReviewsView',
  components: {
    AdminReportManager,
    ReviewFilter,
    ReviewsList,
    AddReviewForm
  },
  setup() {
    // 使用 Pinia auth store
    const authStore = useAuthStore();
    
    // 使用vue-router的useRoute獲取路由參數
    const route = useRoute();
    
    // 獲取路由參數中的campSiteId，並轉為數字
    const campSiteId = ref(parseInt(route.params.campSiteId) || 1);
    
    // 管理員視圖狀態
    const adminView = ref('reviews'); // 'reviews' 或 'reports'
    
    // 篩選條件
    const filters = reactive({
      keyword: '',
      sortBy: 'createdAt',
      direction: 'DESC',
      minRating: 0
    });
    
    // 評價總數
    const totalReviews = ref(0);
    
    // 模態視窗引用
    const addReviewModal = ref(null);
    const handleReportModal = ref(null);
    
    // ReviewsList 組件引用
    const reviewsListRef = ref(null);
    
    // 從 authStore 獲取用戶信息和角色的計算屬性
    const isLoggedIn = computed(() => authStore.isLoggedIn);
    
    // 更精確的角色判斷
    const userRole = computed(() => authStore.role);
    
    // 是否為超級管理員
    const isSuperAdmin = computed(() => userRole.value === 'SUPER_ADMIN');
    
    // 是否為一般管理員
    const isAdmin = computed(() => userRole.value === 'ADMIN');
    
    // 是否為營地擁有者
    const isCampOwner = computed(() => userRole.value === 'CAMP_OWNER' || userRole.value === 'owner');
    
    // 是否為普通用戶
    const isRegularUser = computed(() => {
      return userRole.value === 'USER' || userRole.value === 'user' || 
             userRole.value === 'camper' || !userRole.value;
    });
    
    // 用戶ID和用戶名
    const userId = computed(() => authStore.user?.id || null);
    const username = computed(() => authStore.username || '');
    
    // 構建完整的當前用戶對象 - 提供給子組件使用
    const currentUserObject = computed(() => {
      if (!isLoggedIn.value) {
        return {
          id: null,
          username: 'guest',
          role: 'guest'
        };
      }
      
      return {
        id: authStore.user?.id,
        username: authStore.username,
        role: authStore.role,
        fullName: authStore.fullName
      };
    });
    
    // 檢舉相關
    const selectedReport = ref(null);
    const reportHandlerNote = ref('');
    
    // 檢舉類型名稱
    const reportTypes = {
      1: '不實內容',
      2: '冒犯性內容或語言',
      3: '廣告或垃圾訊息',
      4: '暴力或仇恨言論',
      5: '侵犯隱私權',
      6: '其他'
    };
    
    const getReportTypeName = (type) => {
      return reportTypes[type] || '未知';
    };
    
    // 應用篩選條件
    const applyFilters = (newFilters) => {
      Object.assign(filters, newFilters);
    };
    
    // 更新評價總數
    const updateTotalReviews = (count) => {
      totalReviews.value = count;
    };
    
    // 直接操作模態視窗的方法
    const openAddReviewModal = () => {
      // 直接使用 Bootstrap 的 Modal 靜態方法
      const modalElement = document.getElementById('addReviewModal');
      if (modalElement) {
        const modal = new Modal(modalElement);
        modal.show();
        // 保存引用以便稍後關閉
        addReviewModal.value = modal;
      } else {
        console.error('找不到新增評價模態視窗元素');
      }
    };

    const closeAddReviewModal = () => {
      const modalElement = document.getElementById('addReviewModal');
      if (modalElement) {
        // 嘗試用兩種方式關閉模態視窗
        if (addReviewModal.value) {
          try {
            addReviewModal.value.hide();
          } catch (e) {
            console.log('使用實例關閉模態視窗失敗', e);
          }
        }
        // 作為備份，嘗試使用 Bootstrap 靜態方法
        try {
          const modalInstance = Modal.getInstance(modalElement);
          if (modalInstance) {
            modalInstance.hide();
          }
        } catch (e) {
          console.log('使用靜態方法關閉模態視窗失敗', e);
        }
      }
    };
    
    // 處理新增評價按鈕點擊
    const handleAddReviewClick = () => {
      if (!isLoggedIn.value) {
        alert('請先登入後才能發表評價');
        return;
      }
      
      if (isAdmin.value || isSuperAdmin.value) {
        alert('管理員不能發表評價');
        return;
      }
      
      if (isCampOwner.value) {
        alert('營地擁有者不能對營地發表評價');
        return;
      }
      
      openAddReviewModal();
    };
    
    // 處理新增的評價
    const handleReviewAdded = () => {
      console.log('評價已添加，準備關閉模態視窗');
      
      // 關閉模態視窗
      closeAddReviewModal();
      
      // 顯示成功通知
      alert('評價已成功添加！');
      
      // 確保當前視圖為評價系統
      localStorage.setItem('currentView', 'reviews');
      
      // 刷新評價列表
      if (reviewsListRef.value) {
        if (typeof reviewsListRef.value.fetchReviews === 'function') {
          setTimeout(() => {
            reviewsListRef.value.fetchReviews();
          }, 100);
        }
      }
    };
    
    // 處理點讚/取消點讚
    const handleToggleLike = async (reviewId) => {
      if (!isLoggedIn.value) {
        alert('請先登入後才能點讚評價');
        return;
      }
      
      try {
        console.log(`切換評價 ${reviewId} 的讚狀態`);
        
        // 這裡會連接到ReviewsList組件內部的處理邏輯，通常不需要在這裡額外處理
      } catch (error) {
        console.error('點讚操作失敗:', error);
        alert('操作失敗，請稍後再試');
      }
    };
    
    // 處理檢舉 - 使用與新增評價相同的直接操作方式
    const handleReport = (report) => {
      if (!isLoggedIn.value) {
        alert('請先登入後才能處理檢舉');
        return;
      }
      
      if (!isAdmin.value && !isSuperAdmin.value && report.action === 'process') {
        alert('只有管理員才能處理檢舉');
        return;
      }
      
      selectedReport.value = report;
      reportHandlerNote.value = '';
      
      const modalElement = document.getElementById('handleReportModal');
      if (modalElement) {
        const modal = new Modal(modalElement);
        modal.show();
        // 保存引用以便稍後關閉
        handleReportModal.value = modal;
      } else {
        console.error('找不到處理檢舉模態視窗元素');
      }
    };
    
    // 關閉檢舉模態視窗
    const closeReportModal = () => {
      const modalElement = document.getElementById('handleReportModal');
      if (modalElement) {
        // 嘗試用兩種方式關閉模態視窗
        if (handleReportModal.value) {
          try {
            handleReportModal.value.hide();
          } catch (e) {
            console.log('使用實例關閉模態視窗失敗', e);
          }
        }
        // 作為備份，嘗試使用 Bootstrap 靜態方法
        try {
          const modalInstance = Modal.getInstance(modalElement);
          if (modalInstance) {
            modalInstance.hide();
          }
        } catch (e) {
          console.log('使用靜態方法關閉模態視窗失敗', e);
        }
      }
    };
    
    // 批准檢舉
    const approveReport = async () => {
      try {
        if (!selectedReport.value) return;
        
        if (!isAdmin.value && !isSuperAdmin.value) {
          alert('只有管理員才能批准檢舉');
          return;
        }
        
        await axiosapi.put(`/api/review-reports/${selectedReport.value.id}/approve-and-remove`, {
          handlerNote: reportHandlerNote.value
        });
        
        // 關閉模態視窗
        closeReportModal();
        selectedReport.value = null;
        
        // 提示成功
        alert('檢舉已批准，內容已移除');
        
        // 確保當前視圖為評價系統
        localStorage.setItem('currentView', 'reviews');
        
        // 刷新評價列表
        if (reviewsListRef.value) {
          if (typeof reviewsListRef.value.fetchReviews === 'function') {
            setTimeout(() => {
              reviewsListRef.value.fetchReviews();
            }, 100);
          }
        }
      } catch (error) {
        console.error('處理檢舉失敗:', error);
        alert('處理失敗，請稍後再試');
      }
    };
    
    // 駁回檢舉
    const rejectReport = async () => {
      try {
        if (!selectedReport.value) return;
        
        if (!isAdmin.value && !isSuperAdmin.value) {
          alert('只有管理員才能駁回檢舉');
          return;
        }
        
        await axiosapi.put(`/api/review-reports/${selectedReport.value.id}/process`, {
          status: 'rejected',
          handlerNote: reportHandlerNote.value
        });
        
        // 關閉模態視窗
        closeReportModal();
        selectedReport.value = null;
        
        // 提示成功
        alert('檢舉已駁回，內容已恢復可見');
        
        // 確保當前視圖為評價系統
        localStorage.setItem('currentView', 'reviews');
        
        // 刷新評價列表
        if (reviewsListRef.value) {
          if (typeof reviewsListRef.value.fetchReviews === 'function') {
            setTimeout(() => {
              reviewsListRef.value.fetchReviews();
            }, 100);
          }
        }
      } catch (error) {
        console.error('處理檢舉失敗:', error);
        alert('處理失敗，請稍後再試');
      }
    };
    
    // 監聽路由參數變化
    watch(() => route.params.campSiteId, (newCampSiteId) => {
      // 更新campSiteId
      campSiteId.value = parseInt(newCampSiteId) || 1;
      console.log('路由參數變化，新的campSiteId:', campSiteId.value);
      
      // 刷新評價列表
      if (reviewsListRef.value && typeof reviewsListRef.value.fetchReviews === 'function') {
        setTimeout(() => {
          reviewsListRef.value.fetchReviews();
        }, 100);
      }
    });
    
    // 在組件掛載時進行初始化
    onMounted(() => {
      // 檢查並嘗試恢復用戶認證狀態
      authStore.checkAndRestoreAuth();
      
      console.log('評價系統初始化');
      console.log('- campSiteId:', campSiteId.value);
      console.log('- 用戶狀態:', isLoggedIn.value ? '已登入' : '未登入');
      console.log('- 用戶角色:', userRole.value);
      
      // 添加調試代碼，確認canAddReview的計算邏輯
      console.log('- canAddReview:', isLoggedIn.value && isRegularUser.value && !isCampOwner.value);
    });
    
    return {
      adminView,
      filters,
      totalReviews,
      selectedReport,
      reportHandlerNote,
      reviewsListRef,
      campSiteId,
      
      // 從 authStore 獲取的用戶信息
      isLoggedIn,
      userRole,
      isAdmin,
      isSuperAdmin,
      isCampOwner,
      isRegularUser,
      userId,
      username,
      currentUserObject,
      
      // 方法
      getReportTypeName,
      applyFilters,
      updateTotalReviews,
      handleAddReviewClick,
      handleReviewAdded,
      handleToggleLike,
      handleReport,
      approveReport,
      rejectReport
    };
  }
};
</script>
  
<style scoped>
/* 管理員控制區塊 */
.admin-controls {
  background-color: white;
  border-radius: 12px;
  padding: 15px 20px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
  margin-bottom: 20px;
}

.admin-title {
  font-size: 1.3rem;
  margin: 0;
  color: var(--forest-dark);
}

.control-buttons .btn {
  position: relative;
  padding: 8px 16px;
  border-width: 0;
  border-radius: 6px;
  font-weight: 500;
  transition: all 0.2s ease;
}

/* 評價管理按鈕 - 深藍色主題 */
.control-buttons .btn-admin-reviews {
  color: #0d6efd;
  background-color: rgba(13, 110, 253, 0.1);
}

.control-buttons .btn-admin-reviews:hover {
  background-color: rgba(13, 110, 253, 0.2);
}

.control-buttons .btn-admin-reviews.active {
  color: white;
  background-color: #0d6efd;
}

/* 檢舉管理按鈕 - 深紅色主題 */
.control-buttons .btn-admin-reports {
  color: #dc3545;
  background-color: rgba(220, 53, 69, 0.1);
}

.control-buttons .btn-admin-reports:hover {
  background-color: rgba(220, 53, 69, 0.2);
}

.control-buttons .btn-admin-reports.active {
  color: white;
  background-color: #dc3545;
}

/* 全局變量 */
:root {
  --forest-dark: #356648;
  --forest-medium: #4d8d6a;
  --forest-light: #8ebba7;
  --forest-ultralight: #e6f4ea;
}

@media (max-width: 768px) {
  .admin-controls {
    padding: 12px 15px;
  }
  
  .admin-controls .d-flex {
    flex-direction: column;
    gap: 10px;
  }
  
  .control-buttons {
    display: flex;
    width: 100%;
  }
  
  .control-buttons .btn {
    flex: 1;
  }
}
</style>