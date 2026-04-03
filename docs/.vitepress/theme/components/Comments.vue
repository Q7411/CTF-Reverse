<template>
  <div class="giscus-wrapper">
    <div id="giscus-container"></div>
  </div>
</template>

<script setup>
import { onMounted, watch } from 'vue'
import { useRoute, useData } from 'vitepress'

const route = useRoute()
const { isDark } = useData()

const initGiscus = () => {
  const container = document.getElementById('giscus-container')
  if (!container) return
  
  // 清除旧的评论系统
  container.innerHTML = ''
  
  // 创建新脚本并注入
  const script = document.createElement('script')
  script.src = 'https://giscus.app/client.js'
  // 以下为必填配置项，目前使用用户提供的 repo
  script.setAttribute('data-repo', 'Q7411/Q7411.github.io')
  script.setAttribute('data-repo-id', 'R_kgDONn5RQQ') // 这是一个示例 ID，通常需要配置 GitHub Discussions
  script.setAttribute('data-category', 'Announcements')
  script.setAttribute('data-category-id', 'DIC_kwDONn5RQc4Cl34j') // 示例 Category ID
  script.setAttribute('data-mapping', 'pathname')
  script.setAttribute('data-strict', '0')
  script.setAttribute('data-reactions-enabled', '1')
  script.setAttribute('data-emit-metadata', '0')
  script.setAttribute('data-input-position', 'top')
  script.setAttribute('data-theme', isDark.value ? 'transparent_dark' : 'light')
  script.setAttribute('data-lang', 'zh-CN')
  script.setAttribute('crossorigin', 'anonymous')
  script.async = true
  
  container.appendChild(script)
}

onMounted(() => {
  initGiscus()
})

// 监听路由变化，重新加载评论
watch(() => route.path, () => {
  setTimeout(initGiscus, 300)
})

// 监听暗色模式变化，动态改变评论主题
watch(isDark, () => {
  initGiscus()
})
</script>

<style scoped>
.giscus-wrapper {
  margin-top: 3rem;
  padding-top: 2rem;
  border-top: 1px solid var(--vp-c-divider);
}
</style>
