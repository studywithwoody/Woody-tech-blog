# Nuxt Content 完整介紹：打造內容驅動的 Nuxt 應用

#Nuxt #Nuxt3 #NuxtContent #CMS #Markdown #Frontend #VitePress

在 **Nuxt 3** 生態系中，**Nuxt Content** 是一個強大的內容管理模組，讓你可以直接使用 **Markdown、YAML、JSON、CSV** 等檔案作為資料來源，無需額外的後端或資料庫。

這篇文章將從實務角度出發，帶你了解 Nuxt Content 的核心功能、使用情境與最佳實踐。

---

## 一、什麼是 Nuxt Content？

Nuxt Content 是 Nuxt 官方維護的模組，專為「內容驅動型網站」設計。它讓你可以：

- 用 **Markdown** 撰寫文章，自動轉換為 Vue 元件
- 內建 **全文搜尋**、**分類篩選**、**排序** 功能
- 支援 **MDC 語法**（Markdown Components），在 Markdown 中直接使用 Vue 元件
- 自動產生 **目錄（TOC）** 與 **導航結構**
- 完美整合 **Nuxt 3** 的 SSR/SSG 架構

簡單來說，它是一個「**檔案即資料庫**」的解決方案。

---

## 二、安裝與基本設定

### 安裝模組

```bash
npx nuxi module add content
```

或手動安裝：

```bash
npm install @nuxt/content
```

### 設定 `nuxt.config.ts`

```ts
export default defineNuxtConfig({
  modules: ['@nuxt/content'],
  content: {
    // 設定選項
    highlight: {
      theme: 'github-dark'  // 程式碼高亮主題
    },
    markdown: {
      toc: {
        depth: 3,
        searchDepth: 3
      }
    }
  }
})
```

---

## 三、建立你的第一篇文章

在專案根目錄建立 `content/` 資料夾，這裡的檔案會自動被 Nuxt Content 解析：

```
content/
├── blog/
│   ├── hello-world.md
│   └── nuxt-tips.md
└── about.md
```

### 文章範例 `content/blog/hello-world.md`

```md
---
title: Hello World
description: 我的第一篇文章
date: 2026-02-07
tags:
  - nuxt
  - 入門
---

# 歡迎來到我的部落格

這是使用 **Nuxt Content** 撰寫的第一篇文章！

## 為什麼選擇 Nuxt Content？

1. 簡單易用
2. 效能優異
3. 開發體驗極佳
```

---

## 四、在頁面中顯示內容

### 使用 `<ContentDoc>` 元件

最簡單的方式是使用內建的 `<ContentDoc>` 元件：

```vue
<!-- pages/blog/[...slug].vue -->
<template>
  <main>
    <ContentDoc />
  </main>
</template>
```

這會自動根據路由 `/blog/hello-world` 對應到 `content/blog/hello-world.md`。

### 使用 `queryContent()` API

如果需要更多控制，可以使用 Composable API：

```vue
<script setup>
const { data: articles } = await useAsyncData('blog-list', () =>
  queryContent('blog')
    .where({ _draft: false })
    .sort({ date: -1 })
    .limit(10)
    .find()
)
</script>

<template>
  <ul>
    <li v-for="article in articles" :key="article._path">
      <NuxtLink :to="article._path">
        {{ article.title }}
      </NuxtLink>
    </li>
  </ul>
</template>
```

---

## 五、MDC 語法：在 Markdown 中使用 Vue 元件

這是 Nuxt Content 最強大的功能之一。你可以在 Markdown 中直接嵌入 Vue 元件：

### 定義元件 `components/content/Alert.vue`

```vue
<template>
  <div class="alert" :class="type">
    <slot />
  </div>
</template>

<script setup>
defineProps({
  type: {
    type: String,
    default: 'info'
  }
})
</script>

<style scoped>
.alert { padding: 1rem; border-radius: 8px; }
.alert.info { background: #e3f2fd; }
.alert.warning { background: #fff3e0; }
.alert.error { background: #ffebee; }
</style>
```

### 在 Markdown 中使用

```md
::alert{type="warning"}
這是一個警告訊息！請注意此操作不可逆。
::
```

這種寫法讓你能在純 Markdown 中實現複雜的互動式內容。

---

## 六、自動產生目錄（TOC）

Nuxt Content 會自動解析文章的標題結構，產生目錄資料：

```vue
<script setup>
const { data: article } = await useAsyncData('article', () =>
  queryContent('/blog/hello-world').findOne()
)
</script>

<template>
  <aside>
    <h3>目錄</h3>
    <ul>
      <li v-for="link in article.body.toc.links" :key="link.id">
        <a :href="`#${link.id}`">{{ link.text }}</a>
      </li>
    </ul>
  </aside>
</template>
```

---

## 七、內建全文搜尋

Nuxt Content 提供簡單的搜尋功能：

```vue
<script setup>
const search = ref('')
const { data: results } = await useAsyncData(
  'search',
  () => queryContent()
    .where({ title: { $contains: search.value } })
    .find(),
  { watch: [search] }
)
</script>

<template>
  <input v-model="search" placeholder="搜尋文章..." />
  <ul>
    <li v-for="item in results" :key="item._path">
      {{ item.title }}
    </li>
  </ul>
</template>
```

---

## 八、與其他 CMS 的比較

| 功能 | Nuxt Content | Strapi | Contentful |
|------|--------------|--------|------------|
| 設定複雜度 | 極低 | 中等 | 中等 |
| 需要後端 | ❌ | ✅ | ✅ (SaaS) |
| 版本控制 | ✅ Git 原生 | 需額外設定 | ❌ |
| 免費額度 | ✅ 完全免費 | 有限 | 有限 |
| Markdown 支援 | ✅ 原生 | 需套件 | 需套件 |
| Vue 元件嵌入 | ✅ MDC | ❌ | ❌ |

對於技術部落格、文件網站、個人作品集，Nuxt Content 幾乎是最佳選擇。

---

## 九、最佳實踐建議

### 1. 善用 Front Matter

在每篇文章開頭定義結構化資料：

```yaml
---
title: 文章標題
description: SEO 描述
image: /images/cover.jpg
date: 2026-02-07
author: Woody
tags: [nuxt, content, markdown]
draft: false
---
```

### 2. 建立一致的資料夾結構

```
content/
├── blog/           # 部落格文章
├── docs/           # 技術文件
├── projects/       # 專案介紹
└── _partials/      # 可重用片段（底線開頭不會產生路由）
```

### 3. 使用 `<ContentRenderer>` 實現客製化

```vue
<template>
  <ContentRenderer :value="article" class="prose" />
</template>
```

---

## 十、適用場景

Nuxt Content 特別適合：

- ✅ 技術部落格
- ✅ 產品文件網站
- ✅ 個人作品集
- ✅ 公司官網的內容頁面
- ✅ 教學課程網站

不太適合：

- ❌ 需要非技術人員即時編輯的專案（建議搭配 Nuxt Studio）
- ❌ 需要複雜權限管理的 CMS
- ❌ 超大規模內容（數萬篇以上）

---

## 總結

Nuxt Content 讓「內容驅動開發」變得簡單又強大。透過 Markdown 撰寫內容、Git 管理版本、Vue 元件增強互動，你可以專注在內容創作上，而不用煩惱後端架構。

如果你正在建立部落格、文件網站或任何以內容為核心的 Nuxt 專案，**Nuxt Content 絕對值得一試**。

---

## 延伸閱讀

- [Nuxt Content 官方文件](https://content.nuxt.com/)
- [MDC 語法指南](https://content.nuxt.com/usage/markdown)
- [Nuxt Studio](https://nuxt.studio/) - 視覺化編輯器
