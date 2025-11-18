# Vue 3 组合式 API 深入理解

## 前言

在开发 RadarLive-CN 项目时，我深入使用了 Vue 3 的组合式 API（Composition API）。相比传统的选项式 API，组合式 API 让代码组织更加灵活，逻辑复用更容易。本文分享我在实际项目中的使用经验和心得体会。

## 什么是组合式 API

组合式 API 是 Vue 3 引入的新特性，它允许我们使用函数式的方式来组织组件逻辑。核心思想是：**将相关的逻辑组织在一起，而不是分散在不同的选项中**。

### 传统选项式 API 的问题

在 Vue 2 中，我们使用选项式 API：

```javascript
export default {
  data() {
    return {
      imageList: [],
      currentIndex: 0,
      isPlaying: false
    }
  },
  computed: {
    currentImage() {
      return this.imageList[this.currentIndex]
    }
  },
  methods: {
    loadData() {
      // 加载数据
    },
    nextImage() {
      // 下一张
    }
  },
  watch: {
    currentIndex(newVal) {
      // 监听变化
    }
  }
}
```

**问题**：
- 相关逻辑分散在 `data`、`computed`、`methods`、`watch` 中
- 大型组件中，查找相关代码需要上下滚动
- 逻辑复用困难，需要使用 mixins（容易命名冲突）

### 组合式 API 的优势

```javascript
import { ref, computed, watch, onMounted } from 'vue'

export default {
  setup() {
    // 所有相关逻辑都在这里
    const imageList = ref([])
    const currentIndex = ref(0)
    const isPlaying = ref(false)
    
    const currentImage = computed(() => {
      return imageList.value[currentIndex.value]
    })
    
    const loadData = async () => {
      // 加载数据
    }
    
    watch(currentIndex, (newVal) => {
      // 监听变化
    })
    
    onMounted(() => {
      loadData()
    })
    
    return {
      imageList,
      currentIndex,
      currentImage,
      isPlaying
    }
  }
}
```

**优势**：
- 相关逻辑集中在一起，易于理解和维护
- 逻辑复用更容易（提取为 composables 函数）
- TypeScript 支持更好
- 代码组织更灵活

## 核心 API 详解

### ref vs reactive

**ref** - 用于基本类型和简单对象：

```javascript
import { ref } from 'vue'

// 基本类型
const count = ref(0)
const name = ref('suxiaoling')

// 访问值需要使用 .value
count.value++  // 1
name.value = 'new name'

// 在模板中自动解包，不需要 .value
// <div>{{ count }}</div>
```

**reactive** - 用于复杂对象：

```javascript
import { reactive } from 'vue'

const state = reactive({
  imageList: [],
  currentIndex: 0,
  isPlaying: false
})

// 直接访问属性，不需要 .value
state.currentIndex++
state.imageList.push(newImage)
```

**选择建议**：
- 简单值（字符串、数字、布尔值）→ 使用 `ref`
- 复杂对象（对象、数组）→ 使用 `reactive` 或 `ref`
- 在模板中使用时，`ref` 会自动解包，`reactive` 不需要解包

### computed - 计算属性

计算属性会自动缓存，只有依赖变化时才重新计算：

```javascript
import { ref, computed } from 'vue'

const imageList = ref([])
const currentIndex = ref(0)

// 计算属性：自动从索引获取当前图像
const currentImage = computed(() => {
  if (imageList.value.length === 0) return null
  return imageList.value[currentIndex.value]
})

// 计算属性：时间列表项（自动格式化）
const timeListItems = computed(() => {
  return imageList.value.map((item, index) => ({
    index,
    timestamp: item.timestamp,
    display: formatTimestamp(item.timestamp)
  })).reverse()
})
```

**特点**：
- 自动缓存，性能更好
- 依赖变化时自动更新
- 可以设置 getter 和 setter

### watch vs watchEffect

**watch** - 需要明确指定监听源：

```javascript
import { ref, watch } from 'vue'

const currentIndex = ref(0)
const imageList = ref([])

// 监听单个源
watch(currentIndex, (newVal, oldVal) => {
  console.log(`索引从 ${oldVal} 变为 ${newVal}`)
  preloadNearbyFrames(newVal)
})

// 监听多个源
watch([currentIndex, imageList], ([newIndex, newList], [oldIndex, oldList]) => {
  // 处理变化
})

// 深度监听对象
watch(imageList, (newVal) => {
  // 处理变化
}, { deep: true })
```

**watchEffect** - 自动追踪依赖：

```javascript
import { ref, watchEffect } from 'vue'

const currentIndex = ref(0)
const imageList = ref([])

// 自动追踪 currentIndex 和 imageList
watchEffect(() => {
  if (imageList.value.length > 0 && currentIndex.value >= 0) {
    preloadNearbyFrames(currentIndex.value)
  }
})
```

**选择建议**：
- 需要明确知道监听什么 → 使用 `watch`
- 需要自动追踪所有依赖 → 使用 `watchEffect`
- 需要访问旧值 → 使用 `watch`

## 实际项目案例

在 RadarLive-CN 项目中，我使用组合式 API 实现了一个复杂的雷达展示组件。以下是核心状态管理部分：

```javascript
import { ref, computed, watch, onMounted, onUnmounted } from 'vue'

export default {
  setup() {
    // 核心状态
    const imageList = ref([])
    const currentIndex = ref(0)
    const radarMode = ref('mosaic')
    const region = ref('NATIONAL')
    const selectedStation = ref('')
    const isPlaying = ref(false)
    const dateRange = ref(null)
    
    // 计算属性：自动从索引获取当前图像
    const currentImage = computed(() => {
      if (imageList.value.length === 0) return null
      return imageList.value[currentIndex.value]
    })
    
    // 计算属性：时间列表项（自动格式化）
    const timeListItems = computed(() => {
      return imageList.value.map((item, index) => ({
        key: `${item.timestamp}-${index}`,
        index,
        timestamp: item.timestamp,
        display: formatTimestamp(item.timestamp)
      })).reverse()
    })
    
    // 监听索引变化，自动预加载
    watch(currentIndex, (newIndex) => {
      preloadNearbyFrames(newIndex)
    })
    
    // 播放控制
    const playInterval = ref(null)
    const startAutoPlay = () => {
      isPlaying.value = true
      playInterval.value = setInterval(() => {
        if (currentIndex.value < imageList.value.length - 1) {
          currentIndex.value++
        } else if (loopEnabled.value) {
          currentIndex.value = 0
        } else {
          stopAutoPlay()
        }
      }, playbackIntervalMs.value)
    }
    
    const stopAutoPlay = () => {
      isPlaying.value = false
      if (playInterval.value) {
        clearInterval(playInterval.value)
        playInterval.value = null
      }
    }
    
    // 组件挂载时加载数据
    onMounted(() => {
      loadData()
    })
    
    // 组件卸载时清理
    onUnmounted(() => {
      stopAutoPlay()
    })
    
    return {
      // 状态
      imageList,
      currentIndex,
      currentImage,
      timeListItems,
      radarMode,
      region,
      selectedStation,
      isPlaying,
      dateRange,
      // 方法
      startAutoPlay,
      stopAutoPlay
    }
  }
}
```

## 逻辑复用 - Composables

组合式 API 最大的优势是可以将逻辑提取为可复用的函数：

```javascript
// composables/useRadarPlayer.js
import { ref, computed, watch } from 'vue'

export function useRadarPlayer(imageList) {
  const currentIndex = ref(0)
  const isPlaying = ref(false)
  const playbackIntervalMs = ref(100)
  
  const currentImage = computed(() => {
    if (imageList.value.length === 0) return null
    return imageList.value[currentIndex.value]
  })
  
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
  
  const startAutoPlay = () => {
    // 播放逻辑
  }
  
  return {
    currentIndex,
    currentImage,
    isPlaying,
    nextImage,
    previousImage,
    startAutoPlay
  }
}
```

在组件中使用：

```javascript
import { useRadarPlayer } from '@/composables/useRadarPlayer'

export default {
  setup() {
    const imageList = ref([])
    const { currentIndex, currentImage, startAutoPlay } = useRadarPlayer(imageList)
    
    return {
      imageList,
      currentIndex,
      currentImage,
      startAutoPlay
    }
  }
}
```

## 实践心得

### 1. ref vs reactive 的选择

- **简单值用 ref**：字符串、数字、布尔值
- **复杂对象用 reactive**：对象、数组（但也可以都用 ref，看个人习惯）
- **在模板中使用时**：`ref` 会自动解包，`reactive` 不需要解包

### 2. computed 的缓存机制

- 计算属性会自动缓存，只有依赖变化时才重新计算
- 适合用于派生状态（从其他状态计算得出的状态）
- 避免在模板中进行复杂计算

### 3. watch 的使用场景

- 需要执行副作用（如 API 调用、DOM 操作）时使用 `watch`
- 需要访问旧值时使用 `watch`
- 需要精确控制监听时机时使用 `watch`

### 4. 逻辑复用

- 将相关逻辑提取为 composables 函数
- 命名规范：`use` 开头（如 `useRadarPlayer`）
- 返回响应式状态和方法，让组件使用

### 5. 代码组织

- 相关逻辑放在一起
- 使用注释分隔不同功能模块
- 保持 `setup` 函数简洁，复杂逻辑提取到 composables

## 总结

Vue 3 的组合式 API 让代码组织更加灵活，逻辑复用更容易。在实际项目中：

- ✅ 使用 `ref` 和 `reactive` 管理状态
- ✅ 使用 `computed` 计算派生状态
- ✅ 使用 `watch` 监听变化并执行副作用
- ✅ 将逻辑提取为 composables 函数，提高复用性
- ✅ 保持代码组织清晰，相关逻辑集中

通过组合式 API，我们可以写出更清晰、更易维护的 Vue 组件。

---

*本文基于 RadarLive-CN 项目的实际开发经验总结*

