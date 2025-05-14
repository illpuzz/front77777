<!-- AddReviewForm.vue - 評價表單 -->
<template>
  <div class="add-review-form">
    <!-- 表單載入中狀態 -->
    <div v-if="loading" class="loading-state text-center py-4">
      <div class="spinner-border text-success" role="status">
        <span class="visually-hidden">載入中...</span>
      </div>
      <p class="mt-3 text-muted">正在準備評價表單...</p>
    </div>
    
    <!-- 表單內容 -->
    <form v-else @submit.prevent="submitReview" class="review-form">
      <!-- 評價內容 -->
      <div class="mb-3">
        <label for="reviewText" class="form-label">您的評價</label>
        <textarea 
          id="reviewText" 
          class="form-control" 
          v-model="reviewData.reviewText" 
          rows="4" 
          placeholder="請分享您對這個營地的體驗和印象..."
          required
        ></textarea>
      </div>
      
      <!-- 評分區塊 -->
      <div class="ratings-section mb-4">
        <h5 class="ratings-title">評分</h5>
        
        <div class="row g-4">
          <!-- 清潔度評分 -->
          <div class="col-md-4">
            <div class="rating-item">
              <label class="rating-label">清潔度</label>
              <div class="stars">
                <span 
                  v-for="n in 5" 
                  :key="`cleanliness-${n}`"
                  :class="['star', { active: n <= reviewData.cleanlinessRating }]"
                  @click="reviewData.cleanlinessRating = n"
                >
                  <i class="bi bi-star-fill"></i>
                </span>
              </div>
              <div class="rating-value">{{ reviewData.cleanlinessRating || 0 }}/5</div>
            </div>
          </div>
          
          <!-- 便利度評分 -->
          <div class="col-md-4">
            <div class="rating-item">
              <label class="rating-label">便利度</label>
              <div class="stars">
                <span 
                  v-for="n in 5" 
                  :key="`convenience-${n}`"
                  :class="['star', { active: n <= reviewData.convenienceRating }]"
                  @click="reviewData.convenienceRating = n"
                >
                  <i class="bi bi-star-fill"></i>
                </span>
              </div>
              <div class="rating-value">{{ reviewData.convenienceRating || 0 }}/5</div>
            </div>
          </div>
          
          <!-- 友善度評分 -->
          <div class="col-md-4">
            <div class="rating-item">
              <label class="rating-label">友善度</label>
              <div class="stars">
                <span 
                  v-for="n in 5" 
                  :key="`friendliness-${n}`"
                  :class="['star', { active: n <= reviewData.friendlinessRating }]"
                  @click="reviewData.friendlinessRating = n"
                >
                  <i class="bi bi-star-fill"></i>
                </span>
              </div>
              <div class="rating-value">{{ reviewData.friendlinessRating || 0 }}/5</div>
            </div>
          </div>
        </div>
        
        <!-- 整體評分 (自動計算) -->
        <div class="overall-rating mt-3">
          <label class="rating-label">整體評分 (自動計算)</label>
          <div class="stars">
            <span 
              v-for="n in 5" 
              :key="`overall-${n}`"
              :class="['star', { active: n <= calculateOverallRating }]"
            >
              <i class="bi bi-star-fill"></i>
            </span>
          </div>
          <div class="rating-value">{{ calculateOverallRating }}/5</div>
        </div>
      </div>
      
      <!-- 優點和缺點 -->
      <div class="row mb-4">
        <div class="col-md-6">
          <div class="mb-3">
            <label for="pros" class="form-label">優點</label>
            <textarea 
              id="pros" 
              class="form-control" 
              v-model="reviewData.pros" 
              rows="3" 
              placeholder="您認為這個營地的優點是什麼？"
            ></textarea>
          </div>
        </div>
        <div class="col-md-6">
          <div class="mb-3">
            <label for="cons" class="form-label">缺點</label>
            <textarea 
              id="cons" 
              class="form-control" 
              v-model="reviewData.cons" 
              rows="3" 
              placeholder="您認為這個營地有什麼可以改進的地方？"
            ></textarea>
          </div>
        </div>
      </div>
      
      <!-- 圖片上傳 -->
      <div class="image-upload-section mb-4">
        <label class="form-label d-block">上傳圖片</label>
        <div class="image-upload-container">
          <!-- 已選擇的圖片預覽 -->
          <div class="selected-images">
            <div 
              v-for="(image, index) in selectedImages" 
              :key="index"
              class="image-preview"
            >
              <img :src="image.url" alt="預覽圖片" />
              <button 
                type="button"
                class="remove-image"
                @click="removeImage(index)"
              >
                <i class="bi bi-x-circle-fill"></i>
              </button>
            </div>
            
            <!-- 上傳按鈕 -->
            <label 
              v-if="selectedImages.length < 5" 
              class="upload-btn"
            >
              <input 
                type="file" 
                accept="image/*" 
                @change="onImageSelected" 
                multiple
                ref="fileInput"
              />
              <div class="upload-icon">
                <i class="bi bi-plus-lg"></i>
              </div>
              <span>上傳圖片</span>
            </label>
          </div>
          <small class="form-text text-muted">
            最多可上傳 5 張圖片，每張大小不超過 5MB
          </small>
        </div>
      </div>
      
      <!-- 提交按鈕 -->
      <div class="d-flex justify-content-end">
        <button type="button" class="btn btn-outline-secondary me-2" @click="cancel">取消</button>
        <button 
          type="submit" 
          class="btn btn-success" 
          :disabled="submitting || !isFormValid"
        >
          <i class="bi" :class="submitting ? 'bi-hourglass-split' : 'bi-check-circle'"></i>
          {{ submitting ? '提交中...' : '提交評價' }}
        </button>
      </div>
    </form>
    
    <!-- 錯誤訊息 -->
    <div v-if="error" class="alert alert-danger mt-3">
      <i class="bi bi-exclamation-triangle-fill me-2"></i>
      {{ error }}
    </div>
  </div>
</template>

<script>
import { ref, reactive, computed, watch, onMounted } from 'vue';
import axiosapi from '@/plugins/axios.js'; // 修改：使用配置好的axios實例

export default {
  name: 'AddReviewForm',
  props: {
    campSiteId: {
      type: [Number, String],
      required: true
    },
    userId: {
      type: [Number, String],
      required: true
    }
  },
  emits: ['review-added'],
  setup(props, { emit }) {
    // 載入狀態
    const loading = ref(true);
    const submitting = ref(false);
    const error = ref('');
    
    // 評價數據
    const reviewData = reactive({
      reviewText: '',
      cleanlinessRating: 0,
      convenienceRating: 0,
      friendlinessRating: 0,
      overallRating: 0,
      pros: '',
      cons: ''
    });
    
    // 圖片上傳
    const fileInput = ref(null);
    const selectedImages = ref([]);
    
    // 自動計算整體評分
    const calculateOverallRating = computed(() => {
      const { cleanlinessRating, convenienceRating, friendlinessRating } = reviewData;
      
      // 如果有任何一項評分為0，則整體評分也為0
      if (!cleanlinessRating || !convenienceRating || !friendlinessRating) {
        return 0;
      }
      
      // 計算平均分數並四捨五入
      const average = (cleanlinessRating + convenienceRating + friendlinessRating) / 3;
      return Math.round(average);
    });
    
    // 監聽整體評分變化，自動更新reviewData
    watch(calculateOverallRating, (newValue) => {
      reviewData.overallRating = newValue;
    });
    
    // 表單驗證
    const isFormValid = computed(() => {
      return (
        reviewData.reviewText.trim() !== '' &&
        reviewData.cleanlinessRating > 0 &&
        reviewData.convenienceRating > 0 &&
        reviewData.friendlinessRating > 0
      );
    });
    
    // 處理圖片選擇
    const onImageSelected = (event) => {
      const files = event.target.files;
      if (!files || !files.length) return;
      
      // 檢查總數限制
      if (selectedImages.value.length + files.length > 5) {
        alert(`最多只能上傳 5 張圖片，目前已選擇 ${selectedImages.value.length} 張`);
        // 清空輸入
        if (fileInput.value) {
          fileInput.value.value = '';
        }
        return;
      }
      
      // 處理每個選擇的文件
      Array.from(files).forEach(file => {
        // 檢查文件大小
        if (file.size > 5 * 1024 * 1024) {
          alert(`文件 ${file.name} 太大，超過 5MB 限制`);
          return;
        }
        
        // 檢查文件類型
        if (!file.type.startsWith('image/')) {
          alert(`文件 ${file.name} 不是圖片格式`);
          return;
        }
        
        // 創建預覽
        const reader = new FileReader();
        reader.onload = (e) => {
          selectedImages.value.push({
            file: file,
            url: e.target.result
          });
        };
        reader.readAsDataURL(file);
      });
      
      // 重置輸入
      if (fileInput.value) {
        fileInput.value.value = '';
      }
    };
    
    // 移除選擇的圖片
    const removeImage = (index) => {
      selectedImages.value.splice(index, 1);
    };
    
    // 取消評價
    const cancel = () => {
      emit('review-added', { canceled: true });
    };
    
    // 提交評價
    const submitReview = async () => {
      // 表單驗證
      if (!isFormValid.value) {
        error.value = '請填寫評價內容並給予所有項目評分';
        return;
      }
      
      // 開始提交
      submitting.value = true;
      error.value = '';
      
      try {
        console.log('準備提交評價:', {
          userId: props.userId,
          campSiteId: props.campSiteId,
          reviewData: reviewData
        });
        
        // 創建提交數據
        const submitData = {
          userId: props.userId,
          campSiteId: props.campSiteId,
          reviewText: reviewData.reviewText,
          cleanlinessRating: reviewData.cleanlinessRating,
          convenienceRating: reviewData.convenienceRating,
          friendlinessRating: reviewData.friendlinessRating,
          overallRating: reviewData.overallRating,
          pros: reviewData.pros,
          cons: reviewData.cons
        };
        
        // 修改：使用axiosapi替代axios，確保使用配置好的帶有認證的實例
        // 提交評價數據
        const response = await axiosapi.post('/api/reviews', submitData);
        console.log('評價提交成功:', response.data);
        
        // 獲取新建評價的ID
        const newReviewId = response.data.id || response.data.reviewId;
        
        // 如果有選擇圖片，則上傳
        if (selectedImages.value.length > 0) {
          console.log(`開始上傳 ${selectedImages.value.length} 張圖片...`);
          
          // 遍歷所有圖片
          for (const image of selectedImages.value) {
            try {
              const formData = new FormData();
              formData.append('file', image.file);
              formData.append('reviewId', newReviewId);
              
              // 修改：使用axiosapi替代axios來上傳圖片
              const uploadResponse = await axiosapi.post('/api/review-images/upload', formData, {
                headers: {
                  'Content-Type': 'multipart/form-data'
                }
              });
              
              console.log('圖片上傳成功:', uploadResponse.data);
            } catch (uploadError) {
              console.error('圖片上傳失敗:', uploadError);
              // 繼續上傳其他圖片
            }
          }
        }
        
        // 所有操作完成，通知父組件
        emit('review-added', { success: true, reviewId: newReviewId });
      } catch (err) {
        console.error('評價提交失敗:', err);
        error.value = err.response?.data?.message || '評價提交失敗，請稍後再試';
      } finally {
        submitting.value = false;
      }
    };
    
    // 組件掛載時
    onMounted(() => {
      console.log('AddReviewForm 組件掛載');
      console.log('- campSiteId:', props.campSiteId);
      console.log('- userId:', props.userId);
      
      // 模擬載入過程
      setTimeout(() => {
        loading.value = false;
      }, 500);
    });
    
    return {
      loading,
      submitting,
      error,
      reviewData,
      fileInput,
      selectedImages,
      calculateOverallRating,
      isFormValid,
      onImageSelected,
      removeImage,
      cancel,
      submitReview
    };
  }
};
</script>

<style scoped>
.add-review-form {
  font-family: 'Noto Sans TC', sans-serif;
}

.loading-state {
  min-height: 200px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.ratings-section {
  padding: 20px;
  background-color: #f8f9fa;
  border-radius: 10px;
}

.ratings-title {
  font-size: 1.1rem;
  margin-bottom: 15px;
  color: var(--forest-dark, #356648);
  font-weight: 600;
}

.rating-item {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.rating-label {
  font-weight: 500;
  margin-bottom: 8px;
  color: var(--forest-dark, #356648);
}

.stars {
  display: flex;
  margin-bottom: 5px;
}

.star {
  font-size: 24px;
  color: #e0e0e0;
  cursor: pointer;
  margin: 0 2px;
  transition: transform 0.1s ease, color 0.2s ease;
}

.star:hover {
  transform: scale(1.2);
}

.star i {
  color: #e0e0e0;
}

.star.active i {
  color: #ffc107;
}

.rating-value {
  font-size: 0.9rem;
  color: #6c757d;
  margin-top: 5px;
}

.overall-rating {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-top: 20px;
  padding-top: 15px;
  border-top: 1px dashed #dee2e6;
}

.image-upload-container {
  border: 2px dashed #ced4da;
  border-radius: 8px;
  padding: 15px;
  background-color: #f8f9fa;
}

.selected-images {
  display: flex;
  flex-wrap: wrap;
  gap: 15px;
  margin-bottom: 10px;
}

.image-preview {
  width: 100px;
  height: 100px;
  border-radius: 8px;
  overflow: hidden;
  position: relative;
  border: 1px solid #dee2e6;
}

.image-preview img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.remove-image {
  position: absolute;
  top: 5px;
  right: 5px;
  background: rgba(0, 0, 0, 0.5);
  border: none;
  width: 24px;
  height: 24px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  color: white;
  cursor: pointer;
  padding: 0;
}

.remove-image:hover {
  background: rgba(220, 53, 69, 0.8);
}

.upload-btn {
  width: 100px;
  height: 100px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  border: 2px dashed #ced4da;
  border-radius: 8px;
  cursor: pointer;
  color: #6c757d;
  transition: all 0.2s ease;
}

.upload-btn:hover {
  border-color: var(--forest-medium, #4d8d6a);
  color: var(--forest-dark, #356648);
  background-color: rgba(53, 102, 72, 0.05);
}

.upload-icon {
  font-size: 24px;
  margin-bottom: 5px;
}

.upload-btn input[type="file"] {
  display: none;
}

@media (max-width: 768px) {
  .ratings-section {
    padding: 15px;
  }
  
  .star {
    font-size: 20px;
  }
}
</style>