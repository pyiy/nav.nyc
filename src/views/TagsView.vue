<template>
  <div class="tags-page">
    <header class="tags-header">
      <router-link to="/" class="back-btn">← 返回首页</router-link>
      <h1>🏷️ 标签云</h1>
    </header>

    <div class="tag-cloud">
      <span
        v-for="tag in sortedTags"
        :key="tag.name"
        class="tag-item"
        :style="{ fontSize: tag.size + 'px', opacity: tag.opacity }"
        :class="{ active: selectedTag === tag.name }"
        @click="selectTag(tag.name)"
      >
        {{ tag.name }}
        <sup class="tag-count">{{ tag.count }}</sup>
      </span>
    </div>

    <div v-if="selectedTag" class="tag-results">
      <h2>
        标签: {{ selectedTag }}
        <span class="result-count">({{ filteredSites.length }} 个站点)</span>
        <button class="clear-btn" @click="selectedTag = ''">清除筛选</button>
      </h2>
      <div class="sites-grid">
        <a
          v-for="site in filteredSites"
          :key="site.id"
          :href="site.url"
          target="_blank"
          rel="noopener noreferrer"
          class="site-card"
        >
          <img :src="site.icon" :alt="site.name" class="site-icon" @error="handleIconError" />
          <div class="site-info">
            <h3>{{ site.name }}</h3>
            <p v-if="site.description">{{ site.description }}</p>
          </div>
        </a>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import { mockData } from '../mock/mock_data.js'

const selectedTag = ref('')

// Collect all tags with counts
const allSites = []
const tagMap = {}

for (const cat of mockData.categories) {
  for (const site of cat.sites) {
    allSites.push(site)
    if (site.tags && site.tags.length > 0) {
      for (const tag of site.tags) {
        if (!tagMap[tag]) tagMap[tag] = { name: tag, count: 0 }
        tagMap[tag].count++
      }
    }
  }
}

// Sort tags by count, compute size
const sortedTags = computed(() => {
  const tags = Object.values(tagMap).sort((a, b) => b.count - a.count)
  const maxCount = tags.length > 0 ? tags[0].count : 1
  const minCount = tags.length > 0 ? tags[tags.length - 1].count : 1
  return tags.map(t => ({
    ...t,
    size: 14 + Math.round(((t.count - minCount) / Math.max(maxCount - minCount, 1)) * 18),
    opacity: 0.6 + ((t.count - minCount) / Math.max(maxCount - minCount, 1)) * 0.4,
  }))
})

// Filter sites by selected tag
const filteredSites = computed(() => {
  if (!selectedTag.value) return []
  return allSites.filter(s => s.tags && s.tags.includes(selectedTag.value))
})

function selectTag(tag) {
  selectedTag.value = selectedTag.value === tag ? '' : tag
}

function handleIconError(e) {
  e.target.src = 'data:image/svg+xml,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24"><text y="20" font-size="20">🌐</text></svg>'
}
</script>

<style scoped>
.tags-page {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
  min-height: 100vh;
  background: var(--bg-color, #f5f5f5);
}

.tags-header {
  display: flex;
  align-items: center;
  gap: 16px;
  margin-bottom: 30px;
}

.tags-header h1 {
  font-size: 24px;
  margin: 0;
}

.back-btn {
  color: var(--text-color, #333);
  text-decoration: none;
  font-size: 14px;
  padding: 6px 12px;
  border-radius: 6px;
  background: var(--card-bg, #fff);
  border: 1px solid var(--border-color, #e0e0e0);
}

.back-btn:hover {
  background: var(--hover-bg, #eee);
}

.tag-cloud {
  display: flex;
  flex-wrap: wrap;
  gap: 12px;
  align-items: center;
  justify-content: center;
  padding: 30px;
  background: var(--card-bg, #fff);
  border-radius: 12px;
  margin-bottom: 30px;
  line-height: 2;
}

.tag-item {
  cursor: pointer;
  padding: 4px 10px;
  border-radius: 6px;
  transition: all 0.2s;
  color: var(--text-color, #333);
  background: var(--tag-bg, #f0f0f0);
  white-space: nowrap;
}

.tag-item:hover,
.tag-item.active {
  background: var(--primary-color, #409eff);
  color: #fff;
  transform: scale(1.05);
}

.tag-item.active {
  box-shadow: 0 2px 8px rgba(64, 158, 255, 0.4);
}

.tag-count {
  font-size: 10px;
  margin-left: 2px;
  opacity: 0.7;
}

.tag-results h2 {
  font-size: 18px;
  margin-bottom: 16px;
  display: flex;
  align-items: center;
  gap: 8px;
}

.result-count {
  font-size: 14px;
  color: var(--text-secondary, #999);
  font-weight: normal;
}

.clear-btn {
  font-size: 12px;
  padding: 4px 10px;
  border: 1px solid var(--border-color, #ddd);
  border-radius: 4px;
  background: var(--card-bg, #fff);
  cursor: pointer;
  color: var(--text-color, #333);
}

.clear-btn:hover {
  background: var(--hover-bg, #eee);
}

.sites-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 12px;
}

.site-card {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 12px;
  background: var(--card-bg, #fff);
  border-radius: 8px;
  text-decoration: none;
  color: var(--text-color, #333);
  border: 1px solid var(--border-color, #e8e8e8);
  transition: all 0.2s;
}

.site-card:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.site-icon {
  width: 36px;
  height: 36px;
  border-radius: 6px;
  object-fit: contain;
}

.site-info h3 {
  font-size: 14px;
  margin: 0 0 4px;
  line-height: 1.3;
}

.site-info p {
  font-size: 12px;
  color: var(--text-secondary, #999);
  margin: 0;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  max-width: 200px;
}

/* Dark mode */
:root.dark .tags-page {
  background: #1a1a2e;
}

:root.dark .tag-cloud {
  background: #16213e;
}

:root.dark .site-card {
  background: #16213e;
  border-color: #2a2a4a;
}

:root.dark .tag-item {
  background: #2a2a4a;
  color: #e0e0e0;
}

:root.dark .back-btn {
  background: #16213e;
  border-color: #2a2a4a;
  color: #e0e0e0;
}
</style>
