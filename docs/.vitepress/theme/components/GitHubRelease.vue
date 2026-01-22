<script setup>
import { ref, onMounted, computed } from 'vue'
import { marked } from 'marked'

const REPO = 'FelixChristian011226/AutoHPMA'
const API_URL = `https://api.github.com/repos/${REPO}/releases/latest`
const RELEASES_URL = `https://github.com/${REPO}/releases`

const release = ref(null)
const loading = ref(true)
const error = ref(null)

onMounted(async () => {
  try {
    const res = await fetch(API_URL)
    if (!res.ok) {
      if (res.status === 403) {
        throw new Error('API 请求限制已达上限，请稍后再试或直接访问 GitHub')
      }
      throw new Error(`请求失败: ${res.status}`)
    }
    release.value = await res.json()
  } catch (e) {
    error.value = e.message
  } finally {
    loading.value = false
  }
})

// Filter to only show Setup files
const setupAssets = computed(() => {
  if (!release.value?.assets) return []
  return release.value.assets.filter(asset => 
    asset.name.toLowerCase().includes('setup')
  )
})

function formatDate(dateStr) {
  return dateStr ? dateStr.split('T')[0] : ''
}

function formatBody(body) {
  if (!body) return ''
  // Remove the first line if it starts with # AutoHPMA
  const cleaned = body.replace(/^# .+\n?/, '').trim()
  // Render markdown to HTML
  return marked(cleaned)
}
</script>

<template>
  <div class="github-release">
    <!-- Loading State -->
    <div v-if="loading" class="loading">
      <div class="spinner"></div>
      <p>正在获取最新版本信息...</p>
    </div>

    <!-- Error State -->
    <div v-else-if="error" class="error">
      <div class="error-icon">⚠️</div>
      <p>{{ error }}</p>
      <a :href="RELEASES_URL" target="_blank" class="fallback-link">
        前往 GitHub Releases 页面查看
      </a>
    </div>

    <!-- Success State -->
    <div v-else-if="release" class="release-info">
      <div class="release-header">
        <div class="version-badge">{{ release.tag_name }}</div>
        <span class="release-date">发布于 {{ formatDate(release.published_at) }}</span>
      </div>

      <div class="download-section">
        <h3>下载链接</h3>
        <div class="download-buttons">
          <a
            v-for="asset in setupAssets"
            :key="asset.id"
            :href="asset.browser_download_url"
            class="download-btn"
            target="_blank"
          >
            <span class="download-icon">📦</span>
            {{ asset.name }}
            <span class="file-size">({{ (asset.size / 1024 / 1024).toFixed(1) }} MB)</span>
          </a>
        </div>
      </div>

      <div class="changelog-section">
        <h3>更新内容</h3>
        <div class="changelog-content vp-doc" v-html="formatBody(release.body)"></div>
      </div>

      <div class="github-link">
        <a :href="release.html_url" target="_blank">
          在 GitHub 上查看此版本 →
        </a>
      </div>
    </div>

    <!-- Fallback -->
    <div class="fallback-notice">
      <p>
        如需查看历史版本，请访问 
        <a :href="RELEASES_URL" target="_blank">GitHub Releases 页面</a>
      </p>
    </div>
  </div>
</template>

<style scoped>
.github-release {
  max-width: 800px;
  margin: 0 auto;
}

.loading {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 40px;
  color: var(--vp-c-text-2);
}

.spinner {
  width: 40px;
  height: 40px;
  border: 3px solid var(--vp-c-divider);
  border-top-color: var(--vp-c-brand-1);
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

.error {
  text-align: center;
  padding: 40px;
  background: var(--vp-c-danger-soft);
  border-radius: 12px;
  margin: 20px 0;
}

.error-icon {
  font-size: 48px;
  margin-bottom: 16px;
}

.error p {
  color: var(--vp-c-danger-1);
  margin-bottom: 16px;
}

.fallback-link {
  display: inline-block;
  padding: 10px 20px;
  background: var(--vp-c-brand-1);
  color: white;
  border-radius: 8px;
  text-decoration: none;
  font-weight: 500;
  transition: background 0.2s;
}

.fallback-link:hover {
  background: var(--vp-c-brand-2);
}

.release-info {
  background: var(--vp-c-bg-soft);
  border-radius: 16px;
  padding: 24px;
  margin: 20px 0;
}

.release-header {
  display: flex;
  align-items: center;
  gap: 16px;
  margin-bottom: 24px;
  flex-wrap: wrap;
}

.version-badge {
  background: linear-gradient(135deg, var(--vp-c-brand-1), var(--vp-c-brand-2));
  color: white;
  padding: 8px 16px;
  border-radius: 20px;
  font-weight: 600;
  font-size: 1.1em;
}

.release-date {
  color: var(--vp-c-text-2);
  font-size: 0.9em;
}

.download-section h3,
.changelog-section h3 {
  margin: 0 0 12px 0;
  font-size: 1.1em;
  color: var(--vp-c-text-1);
}

.download-buttons {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.download-btn {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 12px 16px;
  background: var(--vp-c-bg);
  border: 1px solid var(--vp-c-divider);
  border-radius: 8px;
  text-decoration: none;
  color: var(--vp-c-text-1);
  transition: all 0.2s;
}

.download-btn:hover {
  border-color: var(--vp-c-brand-1);
  background: var(--vp-c-brand-soft);
}

.download-icon {
  font-size: 1.2em;
}

.file-size {
  color: var(--vp-c-text-3);
  font-size: 0.85em;
  margin-left: auto;
}

.changelog-section {
  margin-top: 24px;
  padding-top: 24px;
  border-top: 1px solid var(--vp-c-divider);
}

.changelog-content {
  background: var(--vp-c-bg);
  padding: 16px;
  border-radius: 8px;
  font-size: 0.95em;
  line-height: 1.7;
  color: var(--vp-c-text-2);
}

.github-link {
  margin-top: 20px;
  text-align: right;
}

.github-link a {
  color: var(--vp-c-brand-1);
  text-decoration: none;
  font-size: 0.9em;
}

.github-link a:hover {
  text-decoration: underline;
}

.fallback-notice {
  margin-top: 32px;
  padding: 16px;
  background: var(--vp-c-tip-soft);
  border-radius: 8px;
  text-align: center;
  font-size: 0.9em;
  color: var(--vp-c-text-2);
}

.fallback-notice a {
  color: var(--vp-c-brand-1);
}
</style>
