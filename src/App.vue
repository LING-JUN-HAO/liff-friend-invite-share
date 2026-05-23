<script setup lang="ts">
import { onMounted } from "vue";
import liff from "@line/liff";

const officialAccountUrl = 'https://line.me/R/ti/p/%40723znbzn';

const handleUserLogin = () => {
  if (!liff.isLoggedIn()) {
    liff.login()
    return false;
  }

  return true;
}

const handleShareMessage = async () => {
  if (!liff.isApiAvailable('shareTargetPicker')) {
    window.alert('此裝置不支援分享功能，將關閉 LIFF 視窗。');
    liff.closeWindow();
    return;
  }

  await liff.shareTargetPicker([
    {
      type: 'flex',
      altText: '加入官方帳號',
      contents: {
        type: 'bubble',
        body: {
          type: 'box',
          layout: 'vertical',
          paddingAll: 'none',
          contents: [
            {
              type: 'image',
              url: 'https://res.cloudinary.com/dj4rwmdiu/image/upload/v1779513494/%E8%AD%B7%E7%89%99-01_path7w.png',
              size: 'full',
              aspectMode: 'fit',
              aspectRatio: '1040:768',
              margin: 'xxl',
              action: {
                type: 'uri',
                label: '加入官方帳號',
                uri: officialAccountUrl
              }
            }
          ]
        }
      }
    }
  ]);
  liff.closeWindow();
}

onMounted(async () => {
  try {
    await liff.init({
      liffId: import.meta.env.VITE_LIFF_ID,
    });
    if (!handleUserLogin()) {
      return;
    }

    await handleShareMessage();
  } catch (error) {
    console.error('LIFF 初始化失敗', error);
  }
});
</script>

<template></template>
