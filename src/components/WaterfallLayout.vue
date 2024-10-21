<script setup>
import { ref, onMounted, onBeforeUnmount, nextTick, watchEffect } from "vue";
const props = defineProps({
  items: {
    type: Array,
    required: true,
  },
  columnWidth: {
    type: Number,
    required: true,
  },
  gap: {
    type: Number,
    required: true,
  },
});
const imgList = ref(null);
const columnCount = ref(0);
const columnHeights = ref([]);
const itemPositions = ref([]);
const calculateLayout = () => {
  if (!imgList.value) return;
  const imgListWidth = imgList.value.offsetWidth;
  columnCount.value = Math.floor(
    (imgListWidth + props.gap) / (props.columnWidth + props.gap)
  );
  columnHeights.value = new Array(columnCount.value).fill(0);
  itemPositions.value = [];

  props.items.forEach((_, index) => {
    const minHeight = Math.min(...columnHeights.value);
    const col = columnHeights.value.indexOf(minHeight); //找出高度最短的位置

    itemPositions.value[index] = {
      top: minHeight,
      left: col * (props.columnWidth + props.gap),
    };
    const itemElement = imgList.value.children[index];

    columnHeights.value[col] += itemElement.offsetHeight + props.gap;
  });

  imgList.value.style.height = `${Math.max(...columnHeights.value)}px`;
};

const getItemStyle = (index) => {
  if (!itemPositions.value[index]) return {};
  return {
    position: "absolute",
    top: `${itemPositions.value[index].top}px`,
    left: `${itemPositions.value[index].left}px`,
    width: `${props.columnWidth}px`,
  };
};

const debouncedCalculateLayout = () => {
  setTimeout(calculateLayout, 200);
};

onMounted(() => {
  recalculate;
  window.addEventListener("resize", debouncedCalculateLayout);
});

onBeforeUnmount(() => {
  window.removeEventListener("resize", debouncedCalculateLayout);
});

watchEffect(() => {
  props.items, calculateLayout, { deep: true };
});

// 讓父組件可以觸發重新計算
const recalculate = () => {
  nextTick(calculateLayout);
};

// 暴露方法給父組件
defineExpose({ recalculate });
</script>

<template>
  <div ref="imgList" class="imgList">
    <div
      class="ImgItem"
      v-for="(item, index) of items"
      :style="getItemStyle(index)"
    >
      <slot :item="item" :index="index"></slot>
    </div>
  </div>
</template>