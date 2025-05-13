<template>
  <div class="booking-cart card p-3 shadow-sm">
    <h5 class="mb-3">購買清單</h5>

    <ul class="list-group mb-3">
      <li
        v-for="item in cartItems"
        :key="item.spotId"
        class="list-group-item d-flex justify-content-between align-items-center"
      >
        <div>
          <div class="fw-bold">{{ item.spotName }}</div>
          <small class="text-muted"
            >{{ item.quantity }} 帳 × ${{ item.unitPrice }}</small
          >
        </div>
        <div>
          <span class="text-danger fw-bold">${{ item.subtotal }}</span>
          <button
            class="btn btn-sm btn-outline-danger ms-2"
            @click="remove(item.spotId)"
          >
            ✕
          </button>
        </div>
      </li>
    </ul>

    <div class="d-flex justify-content-between mb-3">
      <span class="fw-bold">總計</span>
      <span class="fw-bold text-danger">${{ total }}</span>
    </div>

    <router-link
      class="btn btn-danger w-100 text-center"
      :to="cartItems.length === 0 ? '' : '/campBooking'"
      :class="{ disabled: cartItems.length === 0 }"
      @click.prevent="cartItems.length === 0 && $event.preventDefault()"
    >
      立即訂位
    </router-link>
  </div>
</template>

<script setup>
import { storeToRefs } from "pinia";
import { useBookingCartStore } from "@/stores/bookingCartStore";

const cartStore = useBookingCartStore();
const { selectedSpots, totalAmount } = storeToRefs(cartStore);

const cartItems = selectedSpots;
const total = totalAmount;

const remove = (spotId) => {
  cartStore.updateSpotQuantity(spotId, { spotId, quantity: 0 });
};

const submitBooking = () => {
  alert("訂位功能尚未實作");
  // 你可以在這裡做 API 呼叫、表單送出等
};
</script>

<style scoped>
.booking-cart {
  max-width: 400px;
  margin-left: auto;
  position: sticky;
  top: 100px;
  background-color: #fff;
  border-radius: 8px;
}
</style>
