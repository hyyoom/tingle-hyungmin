<template>
  <main class="container">
    <StarMenu :id="id" />
    <div class="container">
      <h1>상품목록</h1>
      <RouterLink :to="`/${id}/store/manage`"> 설정 </RouterLink>
      <!-- //////정렬기준 수정할 수 있게 row 하나 추가해서 변경 가능하게 -->
      <!-- //////available = false면 비활성화하고 회색처리  -->
      <div v-if="altProducts" class="tw-grid tw-grid-cols-5 tw-gap-2">
        <div
          v-for="product in altProducts"
          :key="product.productId"
          class="product-card tw-rounded-lg tw-transition tw-mb-5"
        >
          <RouterLink :to="`/${id}/store/${product.productId}`" class="tw-flex tw-flex-col">
            <img :src="product.imageUrl[0].url" alt="" class="tw-w-full tw-h-60 tw-object-cover" />
            <div class="product-info tw-text-left tw-py-1">
              <span class="tw-text-md tw-font-semibold tw-truncate">{{ product.name }}</span>
              <span class="tw-text-lg tw-font-bold tw-text-gray-800">{{
                product.formattedPrice
              }}</span>
            </div>
          </RouterLink>
        </div>
      </div>
      <div v-else>스타의 상품 목록이 없습니다.</div>
    </div>
    <StoreCreate />
  </main>
</template>

<script setup lang="ts">
import { ref, onMounted, computed } from 'vue'
import { RouterLink } from 'vue-router'
import axios from 'axios'

import StarMenu from '@/components/StarMenu/StarMenu.vue'
import StoreCreate from '@/components/StarMenu/Store/StoreCreate.vue'

// 상속
const props = defineProps(['id'])
const id = ref(props.id)

// 전체 상품 출력
import type { Goods } from '@/common/types'
const products = ref<Goods[] | null>(null)
////// props로 받은거 나오게 설정해야함
const starName = 'l2esm24'

const getAllProducts = async (starName: string) => {
  try {
    const response = await axios.get(`http://localhost:8080/product/getByStarName/${starName}`)
    if (response.data.resultCode === 'SUCCESS') {
      products.value = response.data.data
    }
  } catch (error) {
    console.error('스타의 상품 목록 조회 중 오류 발생', error)
  }
}

const altProducts = computed(() => {
  return products.value?.map((product) => ({
    ...product,
    formattedPrice: new Intl.NumberFormat('ko-KR', { style: 'currency', currency: 'KRW' }).format(
      product.price
    )
  }))
})

onMounted(() => {
  getAllProducts(starName)
})

// const selectedProduct = ref<Goods | null>(null);
</script>

<style scoped>
img {
  max-width: 100%;
  margin-bottom: 1rem;
}

.product-card {
  display: flex;
  flex-direction: column;
}

.product-info {
  padding-left: 3px;
  display: flex;
  flex-direction: column;
}

.product-info p {
  margin: 0.5rem 0;
  text-decoration: none;
}
</style>
