<script setup>
import { ref, onMounted, onUnmounted } from "vue";
import WaterfallLayout from "../components/WaterfallLayout.vue";

const myItems = ref([]);
const waterfallRef = ref(null);

const generateImageLinks = (count = 12) => {
  return Array.from({ length: count }, () => ({
    src: `https://picsum.photos/250/${Math.floor(Math.random() * 319) + 250}`,
    loaded: false,
  }));
};

const loadMoreItems = () => {
  const newItems = generateImageLinks();
  myItems.value.push(...newItems);
};

// 初始加載
onMounted(() => {
  loadMoreItems();
});

// 監聽圖片加載完成
const onImageLoad = (index) => {
  myItems.value[index].loaded = true;
  // 當圖片加載完成時，重新計算佈局
  waterfallRef.value?.recalculate();
};

// 檢查是否需要加載更多圖片
const checkLoadMore = () => {
  if (Math.floor(window.scrollY) >= document.body.offsetHeight - 1500) {
    loadMoreItems();
  }
};

onMounted(() => {
  window.addEventListener("scroll", checkLoadMore);
});
onUnmounted(() => {
  window.removeEventListener("scroll", checkLoadMore);
});
</script>

<template>
  <div class="imgBox">
    <WaterfallLayout
      ref="waterfallRef"
      :items="myItems"
      :column-width="250"
      :gap="15"
    >
      <template #default="{ item, index }">
        <div class="image-container">
          <div class="Img">
            <img
              :src="item.src"
              alt="random image"
              @load="onImageLoad(index)"
              :style="{ opacity: item.loaded ? 1 : 0 }"
            />
          </div>
          <div class="Txt"></div>
          <div v-if="!item.loaded" class="placeholder"></div>
        </div>
      </template>
    </WaterfallLayout>
  </div>
</template>
