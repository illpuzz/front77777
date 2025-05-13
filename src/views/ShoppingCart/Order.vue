<template>
    <div>
        <div class="header">🏕️ 訂單管理</div>

        <div class="search-section">
            <h3>🔍 訂單搜尋</h3>
            <div class="search-bar">
                <select v-model="orderType">
                    <option value="product">商品訂單</option>
                    <option value="camp">營區訂單</option>
                </select>
                <input type="text" v-model="userId" placeholder="輸入使用者ID" />
                <input type="date" v-model="date" />
                <button @click="searchOrders">搜尋</button>
            </div>
        </div>

        <div class="container">
            <div class="tab-menu">
                <a href="#" class="filter-link" v-for="status in statusFilters" :key="status.value"
                    :class="{ active: currentFilter === status.value }" @click.prevent="filterOrders(status.value)">
                    {{ status.label }}
                </a>
            </div>

            <div class="table-container">
                <table>
                    <thead>
                        <tr>
                            <th v-for="header in tableHeaders" :key="header">{{ header }}</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr v-for="order in filteredOrders" :key="order.id">
                            <td>{{ order.id }}</td>
                            <td>{{ order.user }}</td>
                            <td>${{ order.total }}</td>
                            <td>{{ order.coupon }}</td>
                            <td>${{ order.payment }}</td>
                            <td>
                                <template v-if="editingOrderId === order.id">
                                    <select v-model="order.method">
                                        <option value="信用卡">信用卡</option>
                                        <option value="貨到付款">貨到付款</option>
                                    </select>
                                </template>
                                <template v-else>
                                    {{ order.method }}
                                </template>
                            </td>
                            <td>
                                <template v-if="editingOrderId === order.id">
                                    <select v-model="order.status">
                                        <option value="已付款">已付款</option>
                                        <option value="未付款">未付款</option>
                                    </select>
                                </template>
                                <template v-else>
                                    {{ order.status }}
                                </template>
                            </td>
                            <td>{{ order.date }}</td>
                            <td>
                                <div class="action-buttons">
                                    <template v-if="editingOrderId === order.id">
                                        <button class="btn-edit" @click="saveOrder(order)">儲存</button>
                                    </template>
                                    <template v-else>
                                        <button class="btn-edit" @click="editOrder(order)">修改</button>
                                    </template>
                                    <button class="btn-delete" @click="deleteOrder(order.id)">刪除</button>
                                </div>
                            </td>
                        </tr>
                    </tbody>
                </table>
                <p class="no-records" v-if="filteredOrders.length === 0">沒有符合條件的訂單</p>
            </div>
        </div>
    </div>
</template>

<script setup>
import { ref, computed, watch, onMounted } from 'vue'
import axiosapi from '@/plugins/axios.js';
import Swal from 'sweetalert2'

const orderType = ref('product')
const userId = ref('')
const date = ref('')
const currentFilter = ref('all')
const orders = ref([])
const searchedUserId = ref('')
const editingOrderId = ref(null)

// 初次載入時預設取商品訂單
onMounted(() => {
    fetchOrders()
})

// 自動偵測 orderType 改變，重新取資料
watch(orderType, () => {
    fetchOrders()
})

const fetchOrders = async () => {
    try {
        if (orderType.value === 'product') {
            const response = await axiosapi.get(`/api/orders`)
            orders.value = response.data.map((order) => ({
                id: order.id,
                user: order.userName,
                total: order.totalprice,
                coupon: order.couponName,
                payment: order.finalprice,
                method: order.paymentMethod,
                status: order.status,
                date: order.createdAt
            }));
        }
        else if (orderType.value === 'camp') {
            orders.value = [{
                id: '1003',
                user: 'John',
                total: '$500',
                coupon: '無',
                payment: '$500',
                method: '貨到付款',
                status: '已取消',
                date: '2025-04-03'
            }];
        }
    } catch (error) {
        console.error('資料取得失敗:', error)
    }
}

const editOrder = (order) => {
    editingOrderId.value = order.id
}

const saveOrder = async (order) => {
    try {
        await axiosapi.put(`/api/orders/${order.id}`, {
            paymentMethod: order.method,
            status: order.status
        })
        editingOrderId.value = null
        Swal.fire('成功', '訂單已更新', 'success')
    } catch (error) {
        console.error('更新失敗:', error)
        Swal.fire('錯誤', '更新失敗，請稍後再試', 'error')
    }
}

const deleteOrder = async (id) => {
    const result = await Swal.fire({
        title: '確定要刪除這筆訂單嗎？',
        text: '刪除後無法恢復！',
        icon: 'warning',
        showCancelButton: true,
        confirmButtonColor: '#e57373',
        cancelButtonColor: '#9e9e9e',
        confirmButtonText: '確定刪除',
        cancelButtonText: '取消'
    });

    if (result.isConfirmed) {
        try {
            await axiosapi.delete(`/api/orders/${id}`)
            orders.value = orders.value.filter(order => order.id !== id)
            await Swal.fire({
                icon: 'success',
                title: '刪除成功',
                text: '訂單已被刪除。'
            });
        } catch (error) {
            console.error('刪除失敗:', error)
            Swal.fire({
                icon: 'error',
                title: '刪除失敗',
                text: '請稍後再試一次。'
            });
        }
    }
}

const tableHeaders = [
    '訂單編號', '使用者', '商品總金額', '優惠券',
    '付款金額', '付款方式', '狀態', '訂單建立時間', '操作'
]

const statusFilters = [
    { value: 'all', label: '所有訂單' },
    { value: '已付款', label: '已付款' },
    { value: '未付款', label: '未付款' }
]

const searchOrders = () => {
    searchedUserId.value = userId.value.trim().toLowerCase()
}

// 🔍 依狀態、使用者ID、日期進行篩選
const filteredOrders = computed(() => {
    return orders.value.filter(order => {
        const matchStatus = currentFilter.value === 'all' || order.status === currentFilter.value
        const matchDate = date.value === '' || order.date === date.value
        const matchUserId = searchedUserId.value === '' || order.user.toLowerCase().includes(searchedUserId.value)
        return matchStatus && matchDate && matchUserId
    })
})

const filterOrders = (status) => {
    currentFilter.value = status
}
</script>

<style scoped>
.header {
    background-color: #81c784;
    color: #fffde7;
    padding: 30px 15px;
    text-align: center;
    font-size: 28px;
    font-weight: bold;
    font-family: 'Segoe UI', 'Microsoft JhengHei', sans-serif;
    letter-spacing: 1px;
}

.container {
    max-width: 1000px;
    margin: 20px auto;
    background: white;
    padding: 20px;
    border-radius: 10px;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}

.tab-menu {
    border-bottom: 2px solid #dee2e6;
    padding-bottom: 10px;
    margin-bottom: 20px;
}

.tab-menu a {
    margin-right: 15px;
    text-decoration: none;
    font-weight: bold;
    color: #6c757d;
    padding-bottom: 5px;
}

.tab-menu a.active {
    color: #388e3c;
    border-bottom: 3px solid #388e3c;
}

table {
    width: 100%;
    border-collapse: collapse;
    margin-top: 10px;
}

th,
td {
    padding: 15px;
    text-align: center;
    border-bottom: 1px solid #c8e6c9;
}

th {
    background-color: #dcedc8;
    font-weight: bold;
}

/* 🔧 建議 7：固定操作欄寬度，避免擠壓 */
th:nth-child(9),
td:nth-child(9) {
    white-space: nowrap;
    width: 90px;
}

.no-records {
    text-align: center;
    padding: 20px;
    color: #666;
}

.search-section {
    background-color: #f1f8e9;
    padding: 15px 20px;
    border-radius: 8px;
    margin-top: 20px;
    margin-bottom: 20px;
    max-width: 800px;
    margin-left: auto;
    margin-right: auto;
}

.search-section h3 {
    margin-top: 0;
    color: #558b2f;
}

.search-bar {
    display: flex;
    flex-wrap: wrap;
    gap: 15px;
    align-items: center;
}

.search-bar input,
.search-bar button,
.search-bar select {
    padding: 10px;
    border-radius: 5px;
    border: 1px solid #ccc;
}

.search-bar input {
    flex: 1;
    min-width: 150px;
}

.search-bar select {
    background-color: #ffffff;
    color: #333;
    cursor: pointer;
    min-width: 150px;
}

.search-bar button {
    background-color: #66bb6a;
    color: white;
    border: none;
    cursor: pointer;
}

.search-bar button:hover {
    background-color: #388e3c;
}

.action-buttons {
    display: flex;
    gap: 8px;
}

.action-buttons button {
    padding: 6px 10px;
    font-size: 14px;
    border: none;
    border-radius: 4px;
    cursor: pointer;
}

.btn-view {
    background-color: #4fc3f7;
    color: white;
}

.btn-edit {
    background-color: #ffb74d;
    color: white;
}

.btn-delete {
    background-color: #e57373;
    color: white;
}

.btn-view:hover {
    background-color: #039be5;
}

.btn-edit:hover {
    background-color: #fb8c00;
}

.btn-delete:hover {
    background-color: #d32f2f;
}
</style>