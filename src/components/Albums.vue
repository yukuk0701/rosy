<!-- eslint-disable vue/multi-word-component-names -->
<script setup lang="ts">
import { ref } from "vue";
import LayerImage from "./LayerImage.vue";
import LoadingGif from "@/assets/image/load-loading.gif";
import Tab from "./atom/Tab.vue";
import Tabs from "./atom/Tabs.vue";
import TabSlot from "./atom/TabSlot.vue";
import { Fade } from "@egjs/flicking-plugins";

const props = defineProps({
  type: String,
});

const showLayerImage = ref(false),
  pickedImage = ref({});
const tabs = [
  { name: "a", title: "SCENE #1" },
  { name: "b", title: "SCENE #2" },
  { name: "c", title: "SCENE #3" },
];
const plugins = [new Fade()];

function onChanged({ index }: { index: number }) {
  activeIndex.value = index;
}
const activeIndex = ref(0);
const flickingContainer = ref(null);

function changeTab(index: number) {
  activeIndex.value = index;
  const flicking: any = flickingContainer.value;
  flicking?.moveTo(index, 500);
}

const showLayer = (index: number, prefix: string) => {
  showLayerImage.value = true;
  pickedImage.value = {
    img: pictureOf(index, prefix),
    is_full: isFullImage(index),
  };
};
const closeLayer = () => (showLayerImage.value = false);
const pictureOf = (index: number, prefix: string) => {
  return new URL(
    `/src/assets/image/albums/${prefix}_${index}.jpg`,
    import.meta.url
  ).href;
};
const isFullImage = (index: number) => {
  return index === 1 || index === 8;
};

const albumCounts = Array.from({ length: 14 }, (v, k) => k + 1);
</script>

<template>
  <Tabs>
    <template #button>
      <div class="tab_buttons">
        <Tab
          v-for="(tab, index) in tabs"
          :key="index"
          :class="['tab_button']"
          :title="tab.title"
          :isActive="index === activeIndex"
          @click="changeTab(index)"
        />
      </div>
    </template>
    <template #content>
      <Flicking-Items
        ref="flickingContainer"
        :plugins="plugins"
        @changed="onChanged"
      >
        <TabSlot v-for="(tab, _index) in tabs" :key="`tab_${tab.name}`">
          <div class="albums">
            <div
              :class="['picture', { full: isFullImage(index) }]"
              v-for="index in albumCounts"
              :key="`pic_${index}`"
            >
              <img
                v-lazy="{
                  src: pictureOf(index, tab.name),
                  loading: LoadingGif,
                }"
                @click="showLayer(index, tab.name)"
              />
            </div>
          </div>
        </TabSlot>
      </Flicking-Items>
    </template>
  </Tabs>

  <LayerImage
    v-if="showLayerImage"
    :image="pickedImage"
    @wheel.prevent
    @touchmove.prevent
    @scroll.prevent
    @close="closeLayer"
  />
</template>

<style scoped>
.albums {
  width: 100%;
  margin-bottom: 20px;
  display: grid;
  grid-template-columns: 1fr 1fr;
  row-gap: 4px;
  column-gap: 4px;
}
.albums .picture {
  height: auto;
  border-radius: 3px;
  transition: 0.7s all;
  cursor: pointer;
  display: flex;
  justify-content: center;
  flex-direction: column;
}
.albums .picture.full {
  grid-column: span 2;
}

.albums img {
  object-fit: contain;
  width: 100%;
}

img[lazy="loading"] {
  width: 30px;
  height: 30px;
  margin: 0 auto;
}

.tab_buttons {
  display: flex;
  justify-content: space-between;
  margin: 20px 16px;
}
</style>
