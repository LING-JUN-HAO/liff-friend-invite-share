<script setup lang="ts">
defineProps<{ show: boolean }>();
defineEmits<{ close: [] }>();
</script>

<template>
  <div v-if="show" class="privacy-overlay" @click.self="$emit('close')">
    <div class="privacy-sheet">
      <h3 class="privacy-heading">我們如何使用你的資料</h3>
      <div class="privacy-section">
        <p class="privacy-label">蒐集什麼？</p>
        <p class="privacy-text">你的 LINE 頭貼、暱稱（可用化名）、鄉鎮市，以及住在或離開南投的時間。</p>
      </div>
      <div class="privacy-section">
        <p class="privacy-label">用在哪裡？</p>
        <p class="privacy-text">只用來在你的手機上製作政波卡。</p>
      </div>
      <div class="privacy-section">
        <p class="privacy-label">會儲存嗎？</p>
        <p class="privacy-text">你填的暱稱、鄉鎮市和年數，只會用來在你的手機產生政波卡，本活動不會另外儲存，也不會提供給第三方。</p>
      </div>
      <div class="privacy-section">
        <p class="privacy-label">分享圖卡</p>
        <p class="privacy-text">是否分享給朋友由你決定。圖片傳出後無法收回。</p>
      </div>
      <div class="privacy-section">
        <p class="privacy-label">加入好友</p>
        <p class="privacy-text">適用 LINE 與本官方帳號的隱私政策。</p>
      </div>
      <button class="btn btn--green" @click="$emit('close')">我知道了</button>
    </div>
  </div>
</template>

<style scoped>
.privacy-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0,0,0,.45);
  display: flex;
  flex-direction: column;
  justify-content: flex-end;
  z-index: 100;
}
.privacy-sheet {
  background: #fff;
  border-radius: 20px 20px 0 0;
  padding: 28px 20px 40px;
  max-height: 80vh;
  overflow-y: auto;
}
.privacy-heading {
  font-size: 16px;
  font-weight: 700;
  margin: 0 0 16px;
}
.privacy-section { margin-bottom: 14px; }
.privacy-label {
  font-size: 14px;
  font-weight: 700;
  color: #2b2b2b;
  margin: 0 0 4px;
}
.privacy-text {
  font-size: 14px;
  color: #555;
  margin: 0;
  line-height: 1.6;
}
.btn {
  display: block;
  width: 100%;
  border: none;
  border-radius: 999px;
  padding: 15px;
  font-size: 16px;
  font-weight: 700;
  cursor: pointer;
}
.btn--green { background: #06C755; color: #fff; }
</style>
