<script setup lang="ts">
import { computed, onMounted, ref } from 'vue';
import liff from '@line/liff';

type Step = 'form' | 'preview' | 'done';
const step = ref<Step>('form');

// Profile from LINE
const avatarUrl = ref('');

// Form data
const nickname = ref('');
const personType = ref<'nantou' | 'other'>('nantou');
const residenceType = ref<'local' | 'away'>('local');
const township = ref('');
const duration = ref('');
const agreedToTerms = ref(false);
const showPrivacyDetail = ref(false);

const NANTOU_TOWNSHIPS = [
  '南投市', '埔里鎮', '草屯鎮', '竹山鎮', '集集鎮',
  '名間鄉', '鹿谷鄉', '中寮鄉', '魚池鄉', '國姓鄉',
  '水里鄉', '信義鄉', '仁愛鄉',
];

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

// duration 顯示：去掉「內」字讓句子自然
const durationDisplay = computed(() => {
  if (duration.value === '1年內') return '1 年';
  if (duration.value === '5年內') return '5 年';
  if (duration.value === '10年以上') return '10 年以上';
  return duration.value; // 一輩子
});

// 卡片副標題：「埔里鎮・政波成員」
const cardSubLabel = computed(() => {
  if (personType.value === 'nantou') {
    return `${township.value || '南投'}・政波成員`;
  }
  return '外縣市・政波成員';
});

// 卡片位置文字（第二行小字）
const locationLabel = computed(() => {
  if (personType.value === 'nantou') {
    if (residenceType.value === 'local') {
      return duration.value ? `住在南投 ${durationDisplay.value}` : '住在南投';
    }
    return duration.value ? `離鄉 ${durationDisplay.value}` : '旅外南投人';
  }
  return '';
});

const canSubmitForm = computed(() => {
  if (!nickname.value.trim()) return false;
  if (isNantou.value && !township.value) return false;
  if (isNantou.value && !duration.value) return false;
  return agreedToTerms.value;
});

// Build LINE Flex Message
const buildFlexCard = () => {
  const liffUrl = `https://liff.line.me/${import.meta.env.VITE_LIFF_ID}#supportcard`;
  const imageContents = avatarUrl.value
    ? [{ type: 'image', url: avatarUrl.value, size: '72px', aspectMode: 'cover', aspectRatio: '1:1', style: 'circle', align: 'center' }]
    : [];
  const locationContents = locationLabel.value
    ? [{ type: 'text', text: locationLabel.value, color: '#666666', size: 'sm', align: 'center' }]
    : [];
  return {
    type: 'flex',
    altText: `${nickname.value} 加入了 #一般改變南投的力量`,
    contents: {
      type: 'bubble',
      body: {
        type: 'box',
        layout: 'vertical',
        paddingAll: '24px',
        alignItems: 'center',
        contents: [
          ...imageContents,
          { type: 'text', text: nickname.value || '政波成員', weight: 'bold', size: 'xl', align: 'center', margin: 'md' },
          { type: 'text', text: cardSubLabel.value, color: '#666666', size: 'sm', align: 'center' },
          ...locationContents,
          { type: 'text', text: '#一般改變南投的力量', color: '#1d3461', weight: 'bold', size: 'md', align: 'center', margin: 'lg' },
          {
            type: 'box',
            layout: 'vertical',
            backgroundColor: '#1d3461',
            cornerRadius: '999px',
            paddingTop: '10px',
            paddingBottom: '10px',
            paddingStart: '24px',
            paddingEnd: '24px',
            margin: 'sm',
            action: { type: 'uri', label: '成為政波，改變南投！', uri: liffUrl },
            contents: [
              { type: 'text', text: '成為政波，改變南投！', color: '#ffffff', weight: 'bold', align: 'center', size: 'sm' },
            ],
          },
          { type: 'text', text: '南投縣長候選人　温世政', color: '#aaaaaa', size: 'xs', align: 'center', margin: 'sm' },
        ],
      },
    },
  };
};

const goToPreview = () => { step.value = 'preview'; };
const backToForm = () => { step.value = 'form'; };

const confirmCard = async () => {
  step.value = 'done';
  if (import.meta.env.DEV) return;
  try {
    await liff.sendMessages([buildFlexCard() as any]);
  } catch {}
};

const sendToChat = async () => {
  if (import.meta.env.DEV) { alert('DEV: 傳到聊天室'); return; }
  try {
    await liff.sendMessages([buildFlexCard() as any]);
  } catch (e) { console.error(e); }
};

const inviteFriends = async () => {
  if (import.meta.env.DEV) { alert('DEV: 揪朋友一塊'); return; }
  try {
    await liff.shareTargetPicker([buildFlexCard() as any]);
  } catch (e) { console.error(e); }
};

const saveImage = async () => {
  const el = document.getElementById('lipp-card-capture');
  if (!el) return;
  try {
    const { default: html2canvas } = await import('html2canvas');
    const canvas = await html2canvas(el, { useCORS: true, scale: 2, backgroundColor: '#dde8f7' });
    const a = document.createElement('a');
    a.href = canvas.toDataURL('image/png');
    a.download = '政波卡.png';
    a.click();
  } catch (e) { console.error('儲存圖片失敗', e); }
};
</script>

<template>
  <div class="lipp-app">

    <!-- ── 隱私說明 overlay ────────────────────────────── -->
    <div v-if="showPrivacyDetail" class="privacy-overlay" @click.self="showPrivacyDetail = false">
      <div class="privacy-sheet">
        <h3 class="privacy-heading">我們如何使用你的資料</h3>
        <div class="privacy-section">
          <p class="privacy-label">蒐集什麼？</p>
          <p class="privacy-text">你的 LINE 頭貼、暱稱（可用化名）、鄉鎮市，以及住在或離開南投的時間。</p>
        </div>
        <div class="privacy-section">
          <p class="privacy-label">用在哪裡？</p>
          <p class="privacy-text">只用在你的手機上，製作政波卡。</p>
        </div>
        <div class="privacy-section">
          <p class="privacy-label">會儲存嗎？</p>
          <p class="privacy-text">頭貼、鄉鎮市和年數，只會用來在你的手機上產生政波卡。本活動不會另外儲存，也不會儲存給第三方。</p>
        </div>
        <div class="privacy-section">
          <p class="privacy-label">分享圖卡</p>
          <p class="privacy-text">是否分享給朋友由你決定。圖片傳送後無法主動收回。</p>
        </div>
        <div class="privacy-section">
          <p class="privacy-label">加入好友</p>
          <p class="privacy-text">適用 LINE 與本官方帳號的隱私政策。</p>
        </div>
        <button class="btn btn--green" @click="showPrivacyDetail = false">我知道了</button>
      </div>
    </div>

    <!-- ── STEP 1: 填寫表單 ─────────────────────────────── -->
    <div v-if="step === 'form'" class="lipp-page">
      <div class="lipp-modal lipp-modal--scroll">

        <!-- LINE 頭貼 -->
        <div class="avatar-row">
          <div class="avatar-wrap">
            <img v-if="avatarUrl" :src="avatarUrl" class="avatar-img" crossorigin="anonymous" />
            <div v-else class="avatar-placeholder"></div>
          </div>
          <span class="avatar-label">已帶入你的 LINE 頭貼</span>
        </div>

        <!-- 暱稱 -->
        <div class="field">
          <label class="field__label">你想怎麼讓大家稱呼你？（最多 10 字，可用化名）</label>
          <div class="input-wrap">
            <input
              v-model="nickname"
              class="field__input"
              maxlength="10"
              placeholder="輸入名稱"
            />
            <span class="char-count">{{ nickname.length }}/10</span>
          </div>
        </div>

        <!-- 你是南投人還是外縣市朋友 -->
        <div class="field">
          <label class="field__label">你是南投人，還是關心南投的朋友？</label>
          <div class="toggle-group">
            <button
              class="toggle-btn"
              :class="{ 'toggle-btn--active': personType === 'nantou' }"
              @click="onPersonTypeChange('nantou')"
            >南投人</button>
            <button
              class="toggle-btn"
              :class="{ 'toggle-btn--active': personType === 'other' }"
              @click="onPersonTypeChange('other')"
            >外縣市民眾</button>
          </div>
        </div>

        <!-- 外縣市民眾特殊提示 -->
        <div v-if="personType === 'other'" class="other-notice">
          <span class="other-notice__icon">✓</span>
          <p>不是南投人也沒關係！你一樣也是改變南投的一份力量。</p>
        </div>

        <!-- 住在南投 / 駐外地（南投人限定） -->
        <div v-if="personType === 'nantou'" class="field">
          <label class="field__label">你在哪裡？</label>
          <div class="toggle-group">
            <button
              class="toggle-btn"
              :class="{ 'toggle-btn--active': residenceType === 'local' }"
              @click="onResidenceTypeChange('local')"
            >住在南投</button>
            <button
              class="toggle-btn"
              :class="{ 'toggle-btn--active': residenceType === 'away' }"
              @click="onResidenceTypeChange('away')"
            >駐外地</button>
          </div>
        </div>

        <!-- 鄉鎮市選擇（南投人限定） -->
        <div v-if="isNantou" class="field">
          <label class="field__label">{{ townshipLabel }}</label>
          <select v-model="township" class="field__select">
            <option value="" disabled>選擇鄉鎮市</option>
            <option v-for="t in NANTOU_TOWNSHIPS" :key="t" :value="t">{{ t }}</option>
          </select>
        </div>

        <!-- 年資（南投人限定） -->
        <div v-if="isNantou" class="field">
          <label class="field__label">{{ durationLabel }}</label>
          <div class="duration-group">
            <button
              v-for="d in ['1年內', '5年內', '10年以上', '一輩子']"
              :key="d"
              class="duration-btn"
              :class="{ 'duration-btn--active': duration === d, 'duration-btn--alone': d === '一輩子' }"
              @click="duration = d"
            >{{ d }}</button>
          </div>
        </div>

        <!-- 政波約同意 -->
        <div class="field">
          <label class="checkbox-label">
            <input type="checkbox" v-model="agreedToTerms" />
            <span>我已閱讀並同意
              <button class="link-btn" @click.prevent="showPrivacyDetail = !showPrivacyDetail">隱私說明</button>
              ，{{ isNantou ? '同意以頭貼、暱稱與區域製作政波卡。' : '同意以頭貼與暱稱製作政波卡。' }}
            </span>
          </label>
        </div>

        <button
          class="btn btn--green"
          :disabled="!canSubmitForm"
          @click="goToPreview"
        >匯入力量</button>
      </div>
    </div>

    <!-- ── STEP 2: 卡片預覽 ─────────────────────────────── -->
    <div v-else-if="step === 'preview'" class="lipp-page">
      <div class="lipp-stage">

        <div class="card" id="lipp-card-capture">
          <img v-if="avatarUrl" :src="avatarUrl" class="card__avatar" crossorigin="anonymous" />
          <div v-else class="card__avatar card__avatar--placeholder"></div>
          <p class="card__name">{{ nickname }}</p>
          <p class="card__sub">{{ cardSubLabel }}</p>
          <p v-if="locationLabel" class="card__location">{{ locationLabel }}</p>
          <p class="card__hashtag">#一般改變南投的力量</p>
          <div class="card__cta">成為政波，改變南投！</div>
          <p class="card__candidate">南投縣長候選人　温世政</p>
        </div>

        <p class="hint-text">確認沒問題，就讓這股力量流出去</p>

        <div class="btn-group btn-group--col">
          <button class="btn btn--green" @click="confirmCard">送出，成為政波</button>
          <button class="btn btn--outline" @click="backToForm">返回修改</button>
        </div>
      </div>
    </div>

    <!-- ── STEP 3: 完成頁 ──────────────────────────────── -->
    <div v-else-if="step === 'done'" class="lipp-page">
      <div class="lipp-stage">

        <div class="card" id="lipp-card-capture">
          <img v-if="avatarUrl" :src="avatarUrl" class="card__avatar" crossorigin="anonymous" />
          <div v-else class="card__avatar card__avatar--placeholder"></div>
          <p class="card__name">{{ nickname || '政波成員' }}</p>
          <p class="card__sub">{{ cardSubLabel }}</p>
          <p v-if="locationLabel" class="card__location">{{ locationLabel }}</p>
          <p class="card__hashtag">#一般改變南投的力量</p>
          <div class="card__cta">成為政波，改變南投！</div>
          <p class="card__candidate">南投縣長候選人　温世政</p>
        </div>

        <p class="hint-text">你的力量已匯聚成流<br/>傳出去，讓希望擴散成波 🌊<br/>長按圖片也可以直接儲存</p>

        <div class="btn-group btn-group--col">
          <button class="btn btn--green" @click="saveImage">儲存圖片</button>
          <button class="btn btn--outline" @click="sendToChat">傳到聊天室</button>
          <button class="btn btn--green" @click="inviteFriends">擴散這一波</button>
        </div>
      </div>
    </div>

  </div>
</template>

<style scoped>
* { box-sizing: border-box; }

.lipp-app {
  min-height: 100vh;
  background: #f0f4f0;
  font-family: 'Noto Sans TC', system-ui, sans-serif;
  color: #2b2b2b;
}

/* ── 通用頁面容器 ── */
.lipp-page {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

/* ── 表單頁（白底） ── */
.lipp-modal {
  flex: 1;
  background: #fff;
  padding: 24px 20px 36px;
  display: flex;
  flex-direction: column;
}
.lipp-modal--scroll { overflow-y: auto; }

/* ── 隱私說明 overlay ── */
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
  text-align: left;
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

/* ── 頭貼（表單頁） ── */
.avatar-row {
  display: flex;
  align-items: center;
  gap: 12px;
  margin-bottom: 20px;
}
.avatar-wrap { flex: 0 0 64px; }
.avatar-img {
  width: 64px;
  height: 64px;
  border-radius: 50%;
  object-fit: cover;
  display: block;
}
.avatar-placeholder {
  width: 64px;
  height: 64px;
  border-radius: 50%;
  background: #d5d9de;
}
.avatar-label { font-size: 14px; color: #555; }

/* ── 表單欄位 ── */
.field { margin-bottom: 20px; }
.field__label {
  display: block;
  font-size: 14px;
  font-weight: 600;
  color: #2b2b2b;
  margin-bottom: 10px;
}
.input-wrap { position: relative; }
.field__input {
  width: 100%;
  border: 1.5px solid #22b357;
  border-radius: 10px;
  padding: 12px 40px 12px 14px;
  font-size: 16px;
  outline: none;
}
.char-count {
  position: absolute;
  right: 12px;
  top: 50%;
  transform: translateY(-50%);
  font-size: 12px;
  color: #aaa;
}
.field__select {
  width: 100%;
  border: 1.5px solid #d5d9de;
  border-radius: 10px;
  padding: 12px 14px;
  font-size: 16px;
  background: #fff;
  outline: none;
  appearance: none;
  background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='12' height='8' viewBox='0 0 12 8'%3E%3Cpath d='M1 1l5 5 5-5' stroke='%23888' stroke-width='1.5' fill='none' stroke-linecap='round'/%3E%3C/svg%3E");
  background-repeat: no-repeat;
  background-position: right 14px center;
}
.field__select:focus { border-color: #22b357; }

/* ── Segmented control ── */
.toggle-group {
  display: flex;
  background: #e9e9eb;
  border-radius: 10px;
  padding: 3px;
  gap: 3px;
}
.toggle-btn {
  flex: 1;
  padding: 10px 0;
  border: none;
  border-radius: 8px;
  background: transparent;
  font-size: 15px;
  font-weight: 500;
  color: #666;
  cursor: pointer;
  transition: all .2s;
}
.toggle-btn--active {
  background: #fff;
  color: #2b2b2b;
  font-weight: 700;
  box-shadow: 0 1px 4px rgba(0,0,0,.12);
}

/* ── 外縣市民眾提示 ── */
.other-notice {
  display: flex;
  align-items: flex-start;
  gap: 10px;
  background: #e8f7ed;
  border-radius: 12px;
  padding: 14px;
  margin-bottom: 20px;
}
.other-notice__icon {
  width: 24px;
  height: 24px;
  border-radius: 50%;
  background: #22b357;
  color: #fff;
  font-size: 14px;
  font-weight: 700;
  display: flex;
  align-items: center;
  justify-content: center;
  flex: 0 0 24px;
}
.other-notice p { margin: 0; font-size: 14px; color: #1a7f3a; line-height: 1.6; }

/* ── 年資選擇 ── */
.duration-group {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}
.duration-btn {
  flex: 1;
  padding: 10px 0;
  border: 1.5px solid #d5d9de;
  border-radius: 999px;
  background: #fff;
  font-size: 14px;
  font-weight: 500;
  color: #2b2b2b;
  cursor: pointer;
  transition: all .15s;
}
.duration-btn--active {
  border-color: #22b357;
  color: #22b357;
  font-weight: 700;
}
.duration-btn--alone {
  flex: 0 0 auto;
  padding-left: 18px;
  padding-right: 18px;
}

/* ── Checkbox ── */
.checkbox-label {
  display: flex;
  align-items: flex-start;
  gap: 8px;
  font-size: 14px;
  color: #555;
  cursor: pointer;
  line-height: 1.5;
}
.checkbox-label input[type="checkbox"] {
  appearance: none;
  -webkit-appearance: none;
  flex: 0 0 20px;
  width: 20px;
  height: 20px;
  border: 2px solid #d5d9de;
  border-radius: 4px;
  background: #fff;
  cursor: pointer;
  margin-top: 1px;
}
.checkbox-label input[type="checkbox"]:checked {
  background: #22b357;
  border-color: #22b357;
}
.link-btn {
  background: none;
  border: none;
  padding: 0;
  color: #22b357;
  text-decoration: underline;
  cursor: pointer;
  font-size: inherit;
}

/* ── 通用按鈕 ── */
.btn {
  display: block;
  width: 100%;
  border: none;
  border-radius: 999px;
  padding: 15px;
  font-size: 16px;
  font-weight: 700;
  cursor: pointer;
  transition: opacity .15s;
}
.btn:disabled { opacity: .4; cursor: not-allowed; }
.btn--green { background: #06C755; color: #fff; }
.btn--outline { background: #fff; color: #444; border: 1.5px solid #d5d9de; }

/* ── 預覽 & 完成 stage ── */
.lipp-stage {
  display: flex;
  flex-direction: column;
  flex: 1;
  min-height: 100vh;
  padding: 20px 20px 32px;
  background: #fff;
}

/* ── 政波卡片（藍底、垂直置中） ── */
.card {
  background: #dde8f7;
  border: none;
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
  text-align: center;
  margin-bottom: 10px;
}
.card__candidate {
  font-size: 12px;
  color: #888;
  margin: 0;
}

/* ── 說明文字 ── */
.hint-text {
  text-align: center;
  color: #888;
  font-size: 13px;
  line-height: 1.7;
  margin: 16px 0;
}

/* ── 按鈕組 ── */
.btn-group--col {
  display: flex;
  flex-direction: column;
  gap: 0;
  margin-top: auto;
  padding-top: 8px;
}
.btn-group--col .btn { margin-top: 12px; }
</style>
