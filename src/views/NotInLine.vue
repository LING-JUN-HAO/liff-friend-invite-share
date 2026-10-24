<script setup lang="ts">
const LIFF_ID = import.meta.env.VITE_LIFF_ID as string;
const gameUrl = `https://liff.line.me/${LIFF_ID}#moongame`;
</script>

<template>
  <div class="not-in-line">
    <div class="not-in-line__icon">💬</div>
    <p class="not-in-line__title">請用 LINE 開啟此頁面</p>
    <p class="not-in-line__desc">此遊戲需在 LINE 應用程式中才能執行</p>
    <a class="not-in-line__btn" :href="gameUrl">用 LINE 開啟</a>
  </div>
</template>

<style scoped>
.not-in-line {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: 100vh;
  padding: 24px;
  text-align: center;
  font-family: 'Noto Sans TC', system-ui, sans-serif;
}
.not-in-line__icon { font-size: 64px; margin-bottom: 16px; }
.not-in-line__title { font-size: 20px; font-weight: 700; color: #2b2b2b; margin: 0 0 8px; }
.not-in-line__desc { font-size: 14px; color: #555; margin: 0 0 24px; }
.not-in-line__btn {
  display: inline-block;
  background: #06C755;
  color: #fff;
  font-size: 16px;
  font-weight: 700;
  padding: 14px 32px;
  border-radius: 999px;
  text-decoration: none;
}
</style>
