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
      type: 'template',
      altText: '加入官方帳號',
      template: {
        type: 'buttons',
        text: '點擊下方按鈕加入官方帳號好友',
        actions: [
          {
            type: 'uri',
            label: '加入好友',
            uri: officialAccountUrl
          }
        ]
      }
    }
  ]);
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
