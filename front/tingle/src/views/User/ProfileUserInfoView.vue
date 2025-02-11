<template>
  <main class="container">
    <ul
      class="mt-4 d-flex justify-content-around nav nav-underline"
      style="padding-top: 50px; padding-bottom: 30px"
    >
      <li class="nav-item">
        <RouterLink
          :to="`/profile/userinfo`"
          class="nav-link router-link-custom"
          :class="{ active: isActive('/userInfo') }"
          >회원 정보</RouterLink
        >
      </li>
      <li class="nav-item">
        <RouterLink
          :to="`/profile/storage`"
          class="nav-link router-link-custom"
          :class="{ active: isActive('/storage') }"
          >보관함</RouterLink
        >
      </li>
      <li class="nav-item">
        <RouterLink
          :to="`/profile/orders`"
          class="nav-link router-link-custom"
          :class="{ active: isActive('/orders') }"
          >주문 목록</RouterLink
        >
      </li>
      <li></li>
      <li></li>
      <li></li>
    </ul>
    <div
      class="container p-5"
      style="display: flex; justify-content: center; align-items: center; height: 85vh"
    >
      <div
        style="width: 500px; height: 650px; position: relative"
        v-if="fanState!.picture"
        class="p-4 d-flex flex-column justify-content-center align-items-center border"
      >
        <img
          class="mb-5 mx-0 profile-pic"
          :src="fanState!.picture"
          alt=""
          style="max-height: 80%; object-fit: contain; width: auto"
        />
        <div
          style="
            position: absolute;
            bottom: 20px;
            left: 0;
            right: 0;
            display: flex;
            justify-content: space-between;
            padding: 0 20px;
          "
        >
          <button style="width: 50%" @click="uploadImage">
            <h2 class="fw-bold hover-text">프로필 수정</h2>
          </button>
          <button style="width: 50%" @click="deleteImage(fanState!.id)">
            <h2 class="fw-bold hover-text">프로필 삭제</h2>
          </button>
        </div>
      </div>
    </div>
  </main>
</template>

<script setup lang="ts">
import { useUserStore } from '@/stores/user'
import axios from 'axios'
import { onMounted } from 'vue'
import { useRoute } from 'vue-router'

const route = useRoute()
const { fanState } = useUserStore()
const fanId = fanState!.id

const isActive = (path: string) => {
  return route.path.includes(path)
}

onMounted(() => {
  if (!fanState?.picture) {
    fanState!.picture = '/image/basic-profile.png'
  }
})

const uploadImage = async () => {
  try {
    // 파일 선택하기
    const file = await selectFile2()
    if (!file) return
    // FormData 생성
    const formData = new FormData()
    formData.append('file', file)
    formData.append('fanId', String(fanId))

    // Axios 요청 보내기
    const response = await axios.post('http://localhost:8080/user/profilePicture', formData, {
      headers: {
        'Content-Type': 'multipart/form-data'
      }
    })
    if (response.data.resultCode === 'SUCCESS') {
      alert('프로필 사진이 업데이트 되었습니다.')
      fanState!.picture = response.data.data
    } else {
      console.log(formData)
      alert('프로필 사진 업데이트에 실패했습니다.')
    }
  } catch (error) {
    console.error('프로필 사진 업로드 실패:', error)
    alert('프로필 사진 업로드 중 오류가 발생했습니다.')
  }
}

// 파일 선택 함수
const selectFile2 = () => {
  return new Promise<File | null>((resolve) => {
    const input = document.createElement('input')
    input.type = 'file'
    input.onchange = () => {
      resolve(input.files ? input.files[0] : null)
    }
    input.click()
  })
}

const deleteImage = async (fanId: number) => {
  try {
    const response = await axios.post(`http://localhost:8080/user/profilePicture/delete/${fanId}`)
    fanState!.picture = response.data.data
    if (response.data.resultCode === 'SUCCESS') {
      alert('프로필 사진이 삭제 되었습니다.')
      fanState!.picture = response.data.data
      console.log(fanState)
    } else {
      console.log(fanState)
      alert('프로필 사진 삭제에 실패했습니다.')
    }
  } catch (error) {
    console.error('이미지 삭제 실패:', error)
  }
}
</script>

<style scoped>
.profile-pic {
  max-height: auto;
  /* 최대 높이 설정 */
  width: 400px;
  /* 너비 자동 조절 */
  object-fit: cover;
  /* 이미지 비율 유지 */
}

.hover-text {
  transition: transform 0.3s ease-in-out; /* transform 속성에 대한 전환 효과 적용 */
}

.hover-text:hover {
  transform: translateX(10px); /* 호버 시 글자를 오른쪽으로 10픽셀 이동 */
  cursor: pointer;
}
</style>
