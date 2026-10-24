<script setup lang="ts">
defineProps<{
  avatarUrl: string;
  nickname: string;
  cardSubLabel: string;
  locationLabel: string;
}>();
</script>

<template>
  <div class="card" id="card-capture">
    <img v-if="avatarUrl" :src="avatarUrl" class="card__avatar" crossorigin="anonymous" />
    <div v-else class="card__avatar card__avatar--placeholder"></div>
    <p class="card__name">{{ nickname }}</p>
    <p class="card__sub">{{ cardSubLabel }}</p>
    <p v-if="locationLabel" class="card__location">{{ locationLabel }}</p>
    <p class="card__hashtag">#一股改變南投的力量</p>
    <div class="card__cta">成為政波，改變南投！</div>
    <p class="card__candidate">南投縣長候選人　温世政</p>
  </div>
</template>

<style scoped>
.card {
  background: #dde8f7;
  border-radius: 20px;
  padding: 28px 24px 24px;
  box-shadow: 0 4px 20px rgba(29,52,97,.12);
  display: flex;
  flex-direction: column;
  align-items: center;
  text-align: center;
}
.card__avatar {
  width: 90px;
  height: 90px;
  border-radius: 50%;
  object-fit: cover;
  display: block;
  margin-bottom: 14px;
  border: 4px solid rgba(255,255,255,.8);
}
.card__avatar--placeholder {
  width: 90px;
  height: 90px;
  border-radius: 50%;
  background: #c5cdd8;
  margin-bottom: 14px;
  border: 4px solid rgba(255,255,255,.8);
}
.card__name {
  font-size: 20px;
  font-weight: 700;
  margin: 0 0 4px;
  color: #1a1a1a;
}
.card__sub {
  font-size: 13px;
  color: #555;
  margin: 0 0 2px;
}
.card__location {
  font-size: 13px;
  color: #555;
  margin: 0;
}
.card__hashtag {
  font-size: 15px;
  font-weight: 700;
  color: #1d3461;
  margin: 16px 0 12px;
}
.card__cta {
  background: #1d3461;
  color: #fff;
  border-radius: 999px;
  padding: 10px 24px;
  font-size: 14px;
  font-weight: 700;
  margin-bottom: 10px;
}
.card__candidate {
  font-size: 12px;
  color: #888;
  margin: 0;
}
</style>
