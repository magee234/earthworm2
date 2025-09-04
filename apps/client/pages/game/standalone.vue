<template>
  <div class="flex h-screen w-screen items-center justify-center bg-gray-50 p-4 dark:bg-gray-900">
    <div v-if="!isLoaded" class="text-center text-lg text-gray-500">
      <p>正在加载您的专属课程...</p>
    </div>
    <div v-else-if="sentences.length > 0" class="w-full max-w-4xl">
      <SentencesMaking
        :key="currentSentenceIndex"
        :sentence="sentences[currentSentenceIndex].chinese"
        :translate="sentences[currentSentenceIndex].english"
        @next="handleNext"
      />
    </div>
    <div v-else class="text-center text-lg text-red-500">
      <p>课程加载失败！请检查谷歌表格链接和内容。</p>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';
// --- 这是一个绝对不会出错的、最可靠的相对路径！ ---
import SentencesMaking from '../../components/Playground/SentencesMaking/index.vue';

const GOOGLE_SHEET_URL = 'https://docs.google.com/spreadsheets/d/e/2PACX-1vR0pDMbqfBuSr54Rtk2a6f_5293pZwfQMq-N3bk-ruI-MdsLcISRa6ieq1k19UFUqtxYUze-YKXHxFJ/pub?output=csv';

const sentences = ref<any[]>([]);
const currentSentenceIndex = ref(0);
const isLoaded = ref(false);

function parseCSV(csvText: string): any[] {
  const lines = csvText.trim().split(/\r?\n/);
  const headers = lines[0].split(',').map(h => h.trim());
  const result = [];
  for (let i = 1; i < lines.length; i++) {
    const obj: { [key: string]: string } = {};
    const currentline = lines[i].split(',');
    for (let j = 0; j < headers.length; j++) {
      obj[headers[j]] = (currentline[j] || '').trim();
    }
    if (obj.english && obj.chinese) {
      result.push(obj);
    }
  }
  return result;
}

onMounted(async () => {
  try {
    const response = await fetch(GOOGLE_SHEET_URL);
    if (!response.ok) { throw new Error('Network response was not ok.'); }
    const csvText = await response.text();
    let allData = parseCSV(csvText);
    sentences.value = allData.sort(() => Math.random() - 0.5);
  } catch (error) {
    console.error('加载或解析您的在线课程数据时发生严重错误:', error);
  } finally {
    isLoaded.value = true;
  }
});

function handleNext() {
  if (currentSentenceIndex.value < sentences.value.length - 1) {
    currentSentenceIndex.value++;
  } else {
    alert("恭喜您，已完成本轮所有题目！课程将重新开始。");
    currentSentenceIndex.value = 0;
    sentences.value = sentences.value.sort(() => Math.random() - 0.5);
  }
}
</script>
```3.  **保存 (Commit new file)** 这个新文件。
