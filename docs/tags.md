# 🏷️ 标签云

在这里，你可以通过标签快速找到你感兴趣的文章。

<div class="tags-container">
  <a href="/posts/first-post" class="tag-item">Blog</a>
  <a href="/posts/first-post" class="tag-item">Life</a>
  <a href="/CTF-Reverse/BASE64" class="tag-item">CTF</a>
  <a href="/CTF-Reverse/BASE64" class="tag-item">Reverse</a>
  <a href="/CTF-Reverse/BASE64" class="tag-item">Crypto</a>
</div>

<style>
.tags-container {
  display: flex;
  flex-wrap: wrap;
  gap: 12px;
  margin-top: 2rem;
}

.tag-item {
  padding: 8px 16px;
  background-color: var(--vp-c-bg-alt);
  border: 1px solid var(--vp-c-divider);
  border-radius: 20px;
  font-size: 14px;
  font-weight: 500;
  color: var(--vp-c-text-1) !important;
  text-decoration: none !important;
  transition: all 0.3s ease;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
}

.tag-item:hover {
  background-color: var(--vp-c-brand-1);
  color: white !important;
  border-color: var(--vp-c-brand-1);
  transform: translateY(-2px);
  box-shadow: 0 4px 12px var(--vp-c-brand-soft);
}

.dark .tag-item {
  background-color: #1e1e20;
  border-color: rgba(255, 255, 255, 0.1);
}

.dark .tag-item:hover {
  background-color: var(--vp-c-brand-1);
  border-color: var(--vp-c-brand-1);
}
</style>
