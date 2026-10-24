<script setup lang="ts">
import { ref } from 'vue';
import liff from '@line/liff';
import CardDisplay from './CardDisplay.vue';
import { shareOrDownload } from '../../utils/download';

const props = defineProps<{
  avatarUrl: string;
  nickname: string;
  cardSubLabel: string;
  locationLabel: string;
}>();


const buildFlexCard = () => {
  const liffId = import.meta.env.VITE_LIFF_ID;
  const liffUrl = liffId
    ? `https://liff.line.me/${liffId}#supportcard`
    : '';

  const nickname = props.nickname?.trim() || '政波成員';

  const imageContents = props.avatarUrl?.startsWith('https://')
    ? [{
        type: 'image',
        url: props.avatarUrl.trim(),
        size: 'md',
        aspectMode: 'cover',
        aspectRatio: '1:1',
        style: 'circle',
        align: 'center',
      }]
    : [];

  const subLabelContents = props.cardSubLabel?.trim()
    ? [{
        type: 'text',
        text: props.cardSubLabel.trim(),
        color: '#666666',
        size: 'sm',
        align: 'center',
      }]
    : [];

  const locationContents = props.locationLabel?.trim()
    ? [{
        type: 'text',
        text: props.locationLabel.trim(),
        color: '#666666',
        size: 'sm',
        align: 'center',
      }]
    : [];

  const buttonContents = liffUrl
    ? [{
        type: 'box',
        layout: 'vertical',
        backgroundColor: '#1d3461',
        cornerRadius: '999px',
        paddingAll: '12px',
        margin: 'sm',
        action: {
          type: 'uri',
          label: '成為政波，改變南投！',
          uri: liffUrl,
        },
        contents: [{
          type: 'text',
          text: '成為政波，改變南投！',
          color: '#ffffff',
          weight: 'bold',
          align: 'center',
          size: 'sm',
        }],
      }]
    : [];

  return {
    type: 'flex',
    altText: `${nickname} 加入了 #一般改變南投的力量`,
    contents: {
      type: 'bubble',
      body: {
        type: 'box',
        layout: 'vertical',
        paddingAll: '24px',
        alignItems: 'center',
        contents: [
          ...imageContents,
          {
            type: 'text',
            text: nickname,
            weight: 'bold',
            size: 'xl',
            align: 'center',
            margin: 'md',
            wrap: true,
          },
          ...subLabelContents,
          ...locationContents,
          {
            type: 'text',
            text: '#一般改變南投的力量',
            color: '#1d3461',
            weight: 'bold',
            size: 'md',
            align: 'center',
            margin: 'lg',
          },
          ...buttonContents,
          {
            type: 'text',
            text: '南投縣長候選人　温世政',
            color: '#aaaaaa',
            size: 'xs',
            align: 'center',
            margin: 'sm',
          },
        ],
      },
    },
  };
};


const buildInviteCard = () => {
  const liffId = import.meta.env.VITE_LIFF_ID;
  const liffUrl = liffId ? `https://liff.line.me/${liffId}#supportcard` : '';
  const nickname = props.nickname?.trim() || '政波成員';

  const imageContents = props.avatarUrl?.startsWith('https://')
    ? [{
        type: 'image',
        url: props.avatarUrl.trim(),
        size: 'md',
        aspectMode: 'cover',
        aspectRatio: '1:1',
        style: 'circle',
        align: 'center',
      }]
    : [];

  const locationContents = props.locationLabel?.trim()
    ? [{ type: 'text', text: props.locationLabel.trim(), color: '#666666', size: 'sm', align: 'center' }]
    : [];

  return {
    type: 'flex',
    altText: `${nickname} 邀請你加入政波！`,
    contents: {
      type: 'bubble',
      header: {
        type: 'box',
        layout: 'vertical',
        paddingAll: '16px',
        contents: [
          { type: 'text', text: '我成為政波了，你也一起來，讓希望擴散成波！', wrap: true, size: 'sm' },
          { type: 'text', text: '#一般改變南投的力量', size: 'sm', color: '#1d3461', weight: 'bold', margin: 'sm' },
        ],
      },
      body: {
        type: 'box',
        layout: 'vertical',
        paddingAll: '16px',
        alignItems: 'center',
        contents: [
          ...imageContents,
          { type: 'text', text: nickname, weight: 'bold', size: 'lg', align: 'center', margin: 'md' },
          { type: 'text', text: props.cardSubLabel?.trim() || '', color: '#666666', size: 'sm', align: 'center' },
          ...locationContents,
          { type: 'separator', margin: 'lg' },
          { type: 'text', text: '我也加入政波', weight: 'bold', size: 'md', margin: 'lg' },
          { type: 'text', text: '30 秒做出專屬於你的政波卡', size: 'sm', color: '#666666', margin: 'sm' },
          {
            type: 'button',
            style: 'primary',
            color: '#06C755',
            margin: 'md',
            action: { type: 'uri', label: '我也加入政波', uri: liffUrl },
          },
        ],
      },
    },
  };
};

const saveImage = async () => {
  const el = document.getElementById('card-capture');
  if (!el) return;
  try {
    const { default: html2canvas } = await import('html2canvas');
    const canvas = await html2canvas(el, { useCORS: true, scale: 2, backgroundColor: '#dde8f7' });
    canvas.toBlob(async (blob) => {
      if (!blob) return;
      await shareOrDownload(blob, '政波卡.png');
    }, 'image/png');
  } catch (e) { console.error('儲存圖片失敗', e); }
};

const toastMessage = ref('');
let toastTimer: ReturnType<typeof setTimeout> | null = null;

const showToast = (msg: string) => {
  toastMessage.value = msg;
  if (toastTimer) clearTimeout(toastTimer);
  toastTimer = setTimeout(() => { toastMessage.value = ''; }, 2500);
};

const inviteFriends = async () => {
  if (import.meta.env.DEV) { alert('DEV: 揪朋友一塊'); return; }
  try {
    const result = await liff.shareTargetPicker([buildInviteCard() as any]);
    if (result?.status === 'success') {
      showToast('已成功分享給朋友！');
    } else {
      showToast(`分享狀態：${result?.status ?? '未知'}`);
    }
  } catch (e) {
    showToast(`分享失敗：${String(e)}`);
    console.error(e);
  }
};

const sendToChat = async () => {
  if (import.meta.env.DEV) { alert('DEV: 傳到聊天室'); return; }
  try {
    const context = liff.getContext();
    const chatTypes = ['utou', 'room', 'group', 'square_chat'];
    if (!context || !chatTypes.includes(context.type)) {
      showToast('請從 LINE 聊天室開啟此頁面，才能傳送訊息');
      return;
    }
    await liff.sendMessages([buildFlexCard() as any]);
    showToast('已傳送到聊天室！');
  } catch (e) {
    showToast(`傳送失敗：${String(e)}`);
    console.error(e);
  }
};
</script>

<template>
  <div class="stage">
    <CardDisplay
      :avatar-url="avatarUrl"
      :nickname="nickname"
      :card-sub-label="cardSubLabel"
      :location-label="locationLabel"
    />

    <p class="hint-text">你的力量已匯聚成流<br/>傳出去，讓希望擴散成波 🌊<br/>長按圖片也可以直接儲存</p>

    <div class="btn-group">
      <button class="btn btn--green" @click="saveImage">儲存圖片</button>
      <button class="btn btn--outline" @click="sendToChat">傳到聊天室</button>
      <button class="btn btn--green" @click="inviteFriends">擴散這一波</button>
    </div>

    <Transition name="toast">
      <div v-if="toastMessage" class="toast">{{ toastMessage }}</div>
    </Transition>
  </div>
</template>

<style scoped>
.stage {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
  padding: 20px 20px 32px;
  background: #fff;
}
.hint-text {
  text-align: center;
  color: #888;
  font-size: 13px;
  line-height: 1.7;
  margin: 16px 0;
}
.btn-group {
  display: flex;
  flex-direction: column;
  gap: 0;
  margin-top: auto;
  padding-top: 8px;
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
  transition: opacity .15s;
  margin-top: 12px;
}
.btn--green { background: #06C755; color: #fff; }
.btn--outline { background: #fff; color: #444; border: 1.5px solid #d5d9de; }

.toast {
  position: fixed;
  bottom: 32px;
  left: 16px;
  right: 16px;
  background: rgba(0,0,0,.75);
  color: #fff;
  font-size: 14px;
  padding: 10px 16px;
  border-radius: 12px;
  white-space: pre-wrap;
  word-break: break-all;
  pointer-events: none;
}
.toast-enter-active, .toast-leave-active { transition: opacity .3s; }
.toast-enter-from, .toast-leave-to { opacity: 0; }
</style>
