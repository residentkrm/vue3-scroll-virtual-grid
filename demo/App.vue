<script setup>
import { ref, watch } from 'vue'
import VirtualGrid from '../src/VirtualGrid.vue'
import Post from './Post.vue'

const items = ref([])
const viewType = ref('gridview')
const virtualContainer = ref(null);
const loading = ref(false)

var page = 0
const grid = ref(null)

const loadMore = () => {
  if (loading.value) return;
  loading.value = true
  fetch(`https://dummyapi.io/data/v1/post?page=${page}&limit=40`, {
      headers: {'App-Id': '650ab221880a0d168da91bcb'}
    }).then(response=>response.json())
  .then(data=>{ 
    items.value = items.value.concat(data.data)
    loading.value = false
    page++
  });
}

loadMore()

const ToggleView = () => {
  viewType.value = viewType.value === 'gridview' ? 'tableview' : 'gridview'
  grid.value.reset()
}

</script>

<template>
  <div>
    <h1 class="text-center">Vue3 Scroll Virtual Grid</h1>

    <p class="text-center">
      <button class="btn" @click="ToggleView()">toggle view</button>
    </p>

    <div :class="viewType" v-if="items.length > 0">
      <div class="tableview__head">
        <div class="tableview__column-main">Post</div>
        <div class="tableview__columns">
          <div>Owner</div>
          <div>Date</div>
          <div>Likes</div>
        </div>
      </div>

      <div class="virtual-container" ref="virtualContainer">
        <VirtualGrid
          :data="items"
          ref="grid"
          :container="virtualContainer"
          @onScrollEnd = "loadMore"
          v-if="virtualContainer"
        >
          <template v-slot:default="{ item, index }">
            <Post
              :item="item"
              :index="index"
            />
          </template>
        </VirtualGrid>
      </div>

      <p class="text-center" v-if="loading"> loading... </p>
    </div>
  </div>
</template>

