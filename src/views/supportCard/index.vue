<script setup lang="ts">
import { computed, onMounted, ref, watch } from 'vue';
import liff from '@line/liff';
type Duration = '1y' | '5y' | 'over10y' | 'forever'
import PrivacyOverlay from './PrivacyOverlay.vue';
import FormStep from './FormStep.vue';
import PreviewStep from './PreviewStep.vue';
import DoneStep from './DoneStep.vue';

type Step = 'form' | 'preview' | 'done';
const step = ref<Step>('form');

const avatarUrl = ref('');

// 表單資料
const nickname = ref('');
const personType = ref<'nantou' | 'other'>('nantou');
const residenceType = ref<'local' | 'away'>('local');
const township = ref('');
const duration = ref<Duration | ''>('');
const agreedToTerms = ref(false);
const showPrivacyDetail = ref(false);

const stepTitles: Record<Step, string> = {
  'form': '匯入你的力量',
  'preview': '看你的政波',
  'done': '你成為政波了！',
};

watch(step, (s) => {
  document.title = stepTitles[s];
}, { immediate: true });

onMounted(async () => {
  if (import.meta.env.DEV) return;
  try {
    const profile = await liff.getProfile();
    avatarUrl.value = profile.pictureUrl ?? '';
  } catch {}
});

const isNantou = computed(() => personType.value === 'nantou');

const townshipLabel = computed(() =>
  residenceType.value === 'local' ? '你住在南投哪個鄉鎮市？' : '你的老家在南投哪個鄉鎮市？',
);
const durationLabel = computed(() =>
  residenceType.value === 'local' ? '你在南投生活多久了呢？' : '你離開南投多久了呢？',
);

const onPersonTypeChange = (val: 'nantou' | 'other') => {
  personType.value = val;
  residenceType.value = 'local';
  township.value = '';
  duration.value = '';
};
const onResidenceTypeChange = (val: 'local' | 'away') => {
  residenceType.value = val;
  township.value = '';
  duration.value = '';
};

const durationDisplayMap: Record<Duration, string> = {
  '1y':      '1 年',
  '5y':      '5 年',
  'over10y': '10 年以上',
  'forever': '一輩子',
};
const durationDisplay = computed(() => duration.value ? durationDisplayMap[duration.value] : '');

const cardSubLabel = computed(() => {
  if (personType.value === 'nantou') {
    return `${township.value}・政波成員`;
  }
  return '外縣市・政波成員';
});

const locationLabel = computed(() => {
  if (personType.value !== 'nantou') return '';
  if (residenceType.value === 'local') return `住在南投 ${durationDisplay.value}`;
  return `離鄉 ${durationDisplay.value}`;
});

const canSubmitForm = computed(() => {
  if (!nickname.value.trim()) return false;
  if (isNantou.value && !township.value) return false;
  if (isNantou.value && !duration.value) return false;
  return agreedToTerms.value;
});

const goToPreview = () => { step.value = 'preview'; };
const backToForm = () => { step.value = 'form'; };
const confirmCard = () => { step.value = 'done'; };
</script>

<template>
  <div class="app-wrap">
    <PrivacyOverlay :show="showPrivacyDetail" @close="showPrivacyDetail = false" />

    <FormStep
      v-if="step === 'form'"
      :avatar-url="avatarUrl"
      :nickname="nickname"
      :person-type="personType"
      :residence-type="residenceType"
      :township="township"
      :duration="duration"
      :agreed-to-terms="agreedToTerms"
      :can-submit="canSubmitForm"
      :township-label="townshipLabel"
      :duration-label="durationLabel"
      @update:nickname="nickname = $event"
      @update:person-type="onPersonTypeChange"
      @update:residence-type="onResidenceTypeChange"
      @update:township="township = $event"
      @update:duration="duration = $event"
      @update:agreed-to-terms="agreedToTerms = $event"
      @toggle-privacy="showPrivacyDetail = !showPrivacyDetail"
      @submit="goToPreview"
    />

    <PreviewStep
      v-else-if="step === 'preview'"
      :avatar-url="avatarUrl"
      :nickname="nickname"
      :card-sub-label="cardSubLabel"
      :location-label="locationLabel"
      @confirm="confirmCard"
      @back="backToForm"
    />

    <DoneStep
      v-else-if="step === 'done'"
      :avatar-url="avatarUrl"
      :nickname="nickname"
      :card-sub-label="cardSubLabel"
      :location-label="locationLabel"
    />
  </div>
</template>

<style scoped>
.app-wrap {
  min-height: 100vh;
  background: #f0f4f0;
  font-family: 'Noto Sans TC', system-ui, sans-serif;
  color: #2b2b2b;
}
</style>
