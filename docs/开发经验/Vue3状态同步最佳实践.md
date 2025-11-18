# Vue 3 状态同步最佳实践

## 问题背景

在开发 RadarLive-CN 项目时，我遇到了一个典型的状态管理问题：多个组件（时间列表、滑块、播放控制、图像显示）需要同步当前时间索引，如果状态管理不当，会导致：

- 组件间状态不同步
- 需要手动维护多个状态
- 代码复杂，难以维护
- 容易出现 bug

## 解决方案

使用 Vue 3 的响应式系统和计算属性，实现单一数据源（Single Source of Truth）的状态管理模式。

## 核心思路

1. **单一数据源** - 使用一个 `ref` 统一管理当前索引
2. **计算属性** - 使用 `computed` 自动计算派生状态
3. **响应式更新** - 所有组件自动响应状态变化
4. **集中管理** - 所有状态操作都在一个地方

## 完整实现

### 1. 统一的状态管理

```javascript
import { ref, computed, watch } from 'vue'

export default {
  setup() {
    // 单一数据源：当前索引
    const imageList = ref([])
    const currentIndex = ref(0)
    
    // 计算属性：自动从索引获取当前图像
    const currentImage = computed(() => {
      if (imageList.value.length === 0) return null
      return imageList.value[currentIndex.value]
    })
    
    // 计算属性：时间列表项（自动同步）
    const timeListItems = computed(() => {
      return imageList.value.map((item, index) => ({
        key: `${item.timestamp}-${index}`,
        index,
        timestamp: item.timestamp,
        display: formatTimestamp(item.timestamp)
      })).reverse()
    })
    
    return {
      imageList,
      currentIndex,      // 单一数据源
      currentImage,      // 自动计算
      timeListItems     // 自动计算
    }
  }
}
```

### 2. 组件操作方法

```javascript
// 时间轴滑块变化 → 更新索引
const handleSliderChange = (value) => {
  currentIndex.value = value
}

// 时间列表点击 → 更新索引
const handleTimelineItemClick = (index) => {
  currentIndex.value = index
}

// 播放控制 → 更新索引
const nextImage = () => {
  if (currentIndex.value < imageList.value.length - 1) {
    currentIndex.value++
  }
}

const previousImage = () => {
  if (currentIndex.value > 0) {
    currentIndex.value--
  }
}

// 自动播放时自动更新索引
const startAutoPlay = () => {
  playInterval.value = setInterval(() => {
    if (currentIndex.value < imageList.value.length - 1) {
      currentIndex.value++
    } else if (loopEnabled.value) {
      currentIndex.value = 0  // 循环播放
    } else {
      stopAutoPlay()
    }
  }, playbackIntervalMs.value)
}
```

### 3. 模板中的使用

```vue
<template>
  <!-- 时间列表：自动高亮当前项 -->
  <div class="time-list">
    <button
      v-for="item in timeListItems"
      :key="item.key"
      :class="{ active: item.index === currentIndex }"
      @click="handleTimelineItemClick(item.index)"
    >
      {{ item.display }}
    </button>
  </div>
  
  <!-- 时间轴滑块：双向绑定 -->
  <el-slider
    v-model="currentIndex"
    :max="imageList.length - 1"
    @change="handleSliderChange"
  />
  
  <!-- 当前图像：自动更新 -->
  <div class="image-container">
    <img v-if="currentImage" :src="currentImage.url" />
  </div>
  
  <!-- 播放控制：操作索引 -->
  <div class="playback-controls">
    <button @click="previousImage">上一帧</button>
    <button @click="nextImage">下一帧</button>
    <button @click="startAutoPlay">播放</button>
  </div>
</template>
```

### 4. 自动同步机制

```javascript
// 监听索引变化，自动预加载
watch(currentIndex, (newIndex) => {
  preloadNearbyFrames(newIndex)
})

// 所有组件自动响应 currentIndex 的变化
// - 时间列表自动高亮当前项
// - 滑块自动移动到当前位置
// - 图像自动更新
// - 播放控制自动更新状态
```

## 优势

### 1. 状态同步零延迟

所有组件都基于同一个 `currentIndex`，状态变化立即同步到所有组件。

### 2. 代码简洁

不需要手动同步多个状态，所有逻辑集中在 `setup` 函数中。

### 3. 易于维护

单一数据源让状态管理变得简单，修改状态只需要修改一个地方。

### 4. 类型安全

使用 TypeScript 时，类型推导更准确。

## 实践心得

### 1. 单一数据源原则

- 一个状态只在一个地方定义
- 所有组件都从这个数据源读取
- 所有修改都通过这个数据源

### 2. 计算属性的使用

- 派生状态使用 `computed`，自动缓存
- 避免在模板中进行复杂计算
- 计算属性会自动追踪依赖

### 3. 响应式更新

- 使用 `ref` 和 `reactive` 创建响应式状态
- 所有依赖这个状态的组件会自动更新
- 不需要手动触发更新

### 4. 状态操作集中

- 所有状态操作方法都在 `setup` 中定义
- 通过 `return` 暴露给模板使用
- 保持逻辑清晰，易于理解

## 适用场景

这种状态管理模式适用于：

- 多个组件需要共享状态
- 状态之间有依赖关系
- 需要自动同步的场景
- 复杂的状态管理需求

## 总结

通过单一数据源和计算属性，我们可以实现优雅的状态同步：

- ✅ 使用 `ref` 创建单一数据源
- ✅ 使用 `computed` 计算派生状态
- ✅ 所有组件自动响应状态变化
- ✅ 代码简洁，易于维护

这种方式让状态管理变得简单而强大。

---

*本文基于 RadarLive-CN 项目的实际开发经验总结*

