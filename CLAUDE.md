# LinCMS Vvlog 用户门户开发指南

## 技术栈

- **Vue 3** - 前端框架（组合式 API）
- **Vue Router 4** - 路由管理
- **Vuex 4** - 状态管理
- **Axios** - HTTP 客户端
- **ESLint + Airbnb** - 代码规范
- **Sass/SCSS** - CSS 预处理

## 项目定位

**用户门户前端**，主要面向博客展示、文章阅读、评论互动等用户场景。

## 项目架构

### 目录结构
```
src/
├── api/                      # API 接口
│   ├── request.js            # Axios 配置
│   ├── blog.js               # 博客相关接口
│   ├── article.js            # 文章接口
│   ├── comment.js            # 评论接口
│   └── user.js               # 用户接口
├── components/               # 公共组件
│   ├── ArticleCard.vue       # 文章卡片
│   ├── CommentList.vue       # 评论列表
│   ├── UserAvatar.vue        # 用户头像
│   └── ...                   # 其他组件
├── layouts/                  # 布局组件
│   ├── FrontLayout.vue       # 门户布局
│   ├── ArticleLayout.vue     # 文章详情布局
│   └── ...                   # 其他布局
├── router/                   # 路由配置
│   ├── index.js              # 路由主文件
│   └── modules/              # 路由模块
│       ├── blog.js           # 博客路由
│       ├── article.js        # 文章路由
│       └── user.js           # 用户路由
├── store/                    # Vuex 状态管理
│   ├── index.js              # Store 主文件
│   └── modules/              # 状态模块
│       ├── blog.js           # 博客状态
│       ├── user.js           # 用户状态
│       └── comment.js        # 评论状态
├── styles/                   # 全局样式
│   ├── variables.scss        # 样式变量
│   ├── mixins.scss           # 混合宏
│   └── global.scss           # 全局样式
├── utils/                    # 工具函数
│   ├── auth.js               # 认证工具
│   ├── date.js               # 日期工具
│   └── ...                   # 其他工具
├── views/                    # 页面组件
│   ├── Home/                 # 首页
│   │   ├── Index.vue         # 首页
│   │   ├── Articles.vue      # 文章列表
│   │   └── Categories.vue    # 分类页
│   ├── Article/              # 文章详情
│   │   ├── Detail.vue        # 文章详情
│   │   └── Comments.vue      # 评论区域
│   ├── User/                 # 用户中心
│   │   ├── Profile.vue       # 个人资料
│   │   └── Articles.vue      # 我的文章
│   └── Auth/                 # 认证相关
│       ├── Login.vue         # 登录
│       └── Register.vue      # 注册
├── App.vue                   # 根组件
└── main.js                   # 入口文件
```

## 快速开始

### 环境要求
- Node.js 16+（推荐 Node 18+）
- pnpm 或 npm

### 安装与运行
```bash
# 安装依赖
pnpm install

# 开发模式（热重载）
pnpm run dev
# 或
pnpm run serve

# 构建生产版本
pnpm run build

# 预览生产版本
pnpm run preview

# 运行单元测试
pnpm run test:unit

# 代码检查
pnpm run lint
```

### 访问
- **用户门户**：http://localhost:8080（或下一个可用端口）

## 开发规范

### 代码风格
- 遵循 ESLint Airbnb 配置
- 使用 Vue 3 组合式 API
- 组件命名使用 PascalCase（ArticleCard.vue）
- 文件命名使用 kebab-case（article-card.vue）
- 常量使用 UPPER_SNAKE_CASE

### API 接口调用

#### Axios 配置
```javascript
// api/request.js
import axios from 'axios'

const service = axios.create({
  baseURL: process.env.VUE_APP_API_BASE_URL,
  timeout: 10000
})

// 请求拦截器（可添加 token）
service.interceptors.request.use(
  config => {
    const token = localStorage.getItem('token')
    if (token) {
      config.headers.Authorization = `Bearer ${token}`
    }
    return config
  },
  error => Promise.reject(error)
)

// 响应拦截器
service.interceptors.response.use(
  response => {
    const { code, message, data } = response.data
    if (code === 200) {
      return data
    } else {
      console.error(message)
      return Promise.reject(new Error(message))
    }
  },
  error => {
    console.error(error.message)
    return Promise.reject(error)
  }
)

export default service
```

#### 博客 API
```javascript
// api/blog.js
import request from './request'

export const blogApi = {
  // 获取文章列表
  getArticles(params) {
    return request({
      url: '/blog/articles',
      method: 'get',
      params
    })
  },

  // 获取文章详情
  getArticleDetail(id) {
    return request({
      url: `/blog/articles/${id}`,
      method: 'get'
    })
  },

  // 获取分类列表
  getCategories() {
    return request({
      url: '/blog/categories',
      method: 'get'
    })
  },

  // 获取热门文章
  getHotArticles() {
    return request({
      url: '/blog/hot',
      method: 'get'
    })
  },

  // 搜索文章
  searchArticles(keyword) {
    return request({
      url: '/blog/search',
      method: 'get',
      params: { q: keyword }
    })
  }
}
```

#### 评论 API
```javascript
// api/comment.js
import request from './request'

export const commentApi = {
  // 获取评论列表
  getComments(articleId) {
    return request({
      url: `/blog/articles/${articleId}/comments`,
      method: 'get'
    })
  },

  // 发表评论
  addComment(articleId, content) {
    return request({
      url: `/blog/articles/${articleId}/comments`,
      method: 'post',
      data: { content }
    })
  },

  // 删除评论
  deleteComment(commentId) {
    return request({
      url: `/blog/comments/${commentId}`,
      method: 'delete'
    })
  }
}
```

### 页面开发

#### 首页开发
```vue
<template>
  <div class="home">
    <!-- 轮播图或推荐 -->
    <section class="hero">
      <h1>欢迎来到我的博客</h1>
      <p>分享技术、生活与思考</p>
    </section>

    <!-- 文章列表 -->
    <section class="articles">
      <h2>最新文章</h2>
      <ArticleCard
        v-for="article in articles"
        :key="article.id"
        :article="article"
      />
    </section>

    <!-- 分类导航 -->
    <section class="categories">
      <h2>文章分类</h2>
      <div class="category-list">
        <router-link
          v-for="category in categories"
          :key="category.id"
          :to="`/category/${category.id}`"
          class="category-item"
        >
          {{ category.name }}
        </router-link>
      </div>
    </section>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { blogApi } from '@/api/blog'
import ArticleCard from '@/components/ArticleCard.vue'

const articles = ref([])
const categories = ref([])

const fetchData = async () => {
  try {
    const [articlesRes, categoriesRes] = await Promise.all([
      blogApi.getArticles({ page: 1, size: 10 }),
      blogApi.getCategories()
    ])
    articles.value = articlesRes.items
    categories.value = categoriesRes
  } catch (error) {
    console.error('获取数据失败:', error)
  }
}

onMounted(() => {
  fetchData()
})
</script>

<style lang="scss" scoped>
.home {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;

  .hero {
    text-align: center;
    padding: 60px 0;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    color: white;
    border-radius: 8px;
    margin-bottom: 40px;

    h1 {
      font-size: 48px;
      margin-bottom: 16px;
    }

    p {
      font-size: 20px;
      opacity: 0.9;
    }
  }

  .articles {
    margin-bottom: 40px;

    h2 {
      font-size: 32px;
      margin-bottom: 20px;
    }
  }

  .categories {
    h2 {
      font-size: 32px;
      margin-bottom: 20px;
    }

    .category-list {
      display: flex;
      gap: 16px;
      flex-wrap: wrap;

      .category-item {
        padding: 8px 16px;
        background: #f5f7fa;
        border-radius: 4px;
        transition: all 0.3s;

        &:hover {
          background: #e6f1ff;
          color: #409eff;
        }
      }
    }
  }
}
</style>
```

#### 文章详情页
```vue
<template>
  <div class="article-detail">
    <article class="article-content">
      <h1 class="title">{{ article.title }}</h1>

      <div class="meta">
        <span class="author">作者：{{ article.author }}</span>
        <span class="date">{{ formatDate(article.createdAt) }}</span>
        <span class="views">阅读：{{ article.views }}</span>
      </div>

      <div class="content" v-html="article.content"></div>

      <div class="tags">
        <span
          v-for="tag in article.tags"
          :key="tag"
          class="tag"
        >
          {{ tag }}
        </span>
      </div>
    </article>

    <!-- 评论区域 -->
    <section class="comments">
      <CommentList :article-id="articleId" />
    </section>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue'
import { useRoute } from 'vue-router'
import { blogApi } from '@/api/blog'
import { formatDate } from '@/utils/date'
import CommentList from '@/components/CommentList.vue'

const route = useRoute()
const articleId = computed(() => route.params.id)
const article = ref({})

const fetchArticle = async () => {
  try {
    article.value = await blogApi.getArticleDetail(articleId.value)
  } catch (error) {
    console.error('获取文章详情失败:', error)
  }
}

onMounted(() => {
  fetchArticle()
})
</script>
```

### 状态管理

#### 博客状态
```javascript
// store/modules/blog.js
export default {
  namespaced: true,
  state: {
    articles: [],
    categories: [],
    currentArticle: null,
    pagination: {
      page: 1,
      size: 10,
      total: 0
    }
  },
  mutations: {
    SET_ARTICLES(state, articles) {
      state.articles = articles
    },
    SET_CURRENT_ARTICLE(state, article) {
      state.currentArticle = article
    },
    SET_CATEGORIES(state, categories) {
      state.categories = categories
    },
    SET_PAGINATION(state, pagination) {
      state.pagination = { ...state.pagination, ...pagination }
    }
  },
  actions: {
    async fetchArticles({ commit, state }) {
      const data = await blogApi.getArticles(state.pagination)
      commit('SET_ARTICLES', data.items)
      commit('SET_PAGINATION', {
        total: data.total,
        page: data.page
      })
    },
    async fetchArticle({ commit }, id) {
      const article = await blogApi.getArticleDetail(id)
      commit('SET_CURRENT_ARTICLE', article)
    }
  },
  getters: {
    articles: state => state.articles,
    categories: state => state.categories,
    currentArticle: state => state.currentArticle
  }
}
```

### 路由配置

#### 路由示例
```javascript
// router/index.js
import { createRouter, createWebHistory } from 'vue-router'

const routes = [
  {
    path: '/',
    name: 'Home',
    component: () => import('@/views/Home/Index.vue')
  },
  {
    path: '/articles',
    name: 'Articles',
    component: () => import('@/views/Home/Articles.vue')
  },
  {
    path: '/article/:id',
    name: 'ArticleDetail',
    component: () => import('@/views/Article/Detail.vue')
  },
  {
    path: '/category/:id',
    name: 'Category',
    component: () => import('@/views/Home/Categories.vue')
  },
  {
    path: '/user/profile',
    name: 'Profile',
    component: () => import('@/views/User/Profile.vue'),
    meta: {
      requiresAuth: true
    }
  },
  {
    path: '/login',
    name: 'Login',
    component: () => import('@/views/Auth/Login.vue')
  }
]

const router = createRouter({
  history: createWebHistory(),
  routes
})

// 路由守卫
router.beforeEach((to, from, next) => {
  const token = localStorage.getItem('token')
  if (to.meta.requiresAuth && !token) {
    next('/login')
  } else {
    next()
  }
})

export default router
```

### 样式指南

#### 全局样式变量
```scss
// styles/variables.scss
// 主色调
$primary-color: #409eff;
$success-color: #67c23a;
$warning-color: #e6a23c;
$danger-color: #f56c6c;

// 文字颜色
$text-color: #303133;
$text-regular: #606266;
$text-secondary: #909399;

// 背景颜色
$bg-color: #f5f7fa;
$bg-white: #ffffff;

// 间距
$spacing-xs: 4px;
$spacing-sm: 8px;
$spacing-md: 16px;
$spacing-lg: 24px;
$spacing-xl: 32px;

// 圆角
$border-radius: 4px;
$border-radius-lg: 8px;
```

#### 文章内容样式
```scss
.article-content {
  .title {
    font-size: 36px;
    font-weight: bold;
    color: $text-color;
    margin-bottom: 24px;
    line-height: 1.4;
  }

  .meta {
    display: flex;
    gap: 24px;
    color: $text-secondary;
    font-size: 14px;
    margin-bottom: 32px;
    padding-bottom: 24px;
    border-bottom: 1px solid #e4e7ed;
  }

  .content {
    line-height: 1.8;
    font-size: 16px;
    color: $text-regular;

    h1, h2, h3 {
      margin-top: 32px;
      margin-bottom: 16px;
      color: $text-color;
    }

    p {
      margin-bottom: 16px;
    }

    img {
      max-width: 100%;
      border-radius: $border-radius;
    }

    code {
      background: #f5f7fa;
      padding: 2px 6px;
      border-radius: 4px;
      font-family: 'Courier New', monospace;
    }

    pre {
      background: #f5f7fa;
      padding: 16px;
      border-radius: $border-radius;
      overflow-x: auto;
    }
  }

  .tags {
    display: flex;
    gap: 8px;
    margin-top: 32px;
    padding-top: 24px;
    border-top: 1px solid #e4e7ed;

    .tag {
      padding: 4px 12px;
      background: #ecf5ff;
      color: $primary-color;
      border-radius: 16px;
      font-size: 14px;
    }
  }
}
```

## 测试

### 单元测试
```javascript
// tests/unit/views/Home.spec.js
import { mount } from '@vue/test-utils'
import Home from '@/views/Home/Index.vue'

describe('Home', () => {
  it('renders home page', () => {
    const wrapper = mount(Home)
    expect(wrapper.text()).toContain('欢迎来到我的博客')
  })
})
```

### 运行测试
```bash
# 运行测试
pnpm run test:unit

# 监听模式
pnpm run test:unit -- --watch
```

## 构建与部署

### 构建命令
```bash
# 构建开发版本
pnpm run build:dev

# 构建生产版本
pnpm run build:prod

# 预览构建结果
pnpm run preview
```

### 环境配置
```bash
# .env.development
VUE_APP_API_BASE_URL=https://api.dev.example.com

# .env.production
VUE_APP_API_BASE_URL=https://api.prod.example.com
```

## 常见功能

### 搜索功能
```vue
<template>
  <div class="search">
    <input
      v-model="keyword"
      @keyup.enter="handleSearch"
      placeholder="搜索文章..."
    />
    <button @click="handleSearch">搜索</button>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import { useRouter } from 'vue-router'
import { blogApi } from '@/api/blog'

const router = useRouter()
const keyword = ref('')

const handleSearch = async () => {
  if (!keyword.value.trim()) return

  const results = await blogApi.searchArticles(keyword.value)
  router.push({
    name: 'Articles',
    query: { q: keyword.value }
  })
}
</script>
```

### 评论功能
```vue
<template>
  <div class="comment-box">
    <h3>发表评论</h3>
    <textarea
      v-model="content"
      placeholder="写下你的想法..."
      rows="4"
    />
    <button @click="handleSubmit">发表评论</button>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import { commentApi } from '@/api/comment'

const props = defineProps({
  articleId: {
    type: Number,
    required: true
  }
})

const content = ref('')

const handleSubmit = async () => {
  if (!content.value.trim()) return

  try {
    await commentApi.addComment(props.articleId, content.value)
    content.value = ''
    // 刷新评论列表
    // 触发父组件事件或调用 API
  } catch (error) {
    console.error('发表评论失败:', error)
  }
}
</script>
```

## 性能优化

1. **路由懒加载**：使用动态导入
2. **图片懒加载**：`v-lazy` 或 Intersection Observer
3. **文章分页**：长列表分页加载
4. **代码分割**：按页面分割打包
5. **CDN 加速**：静态资源 CDN

## 相关资源

- **Vue 3 文档**：https://v3.cn.vuejs.org/
- **Vue Router 文档**：https://next.router.vuejs.org/
- **Vuex 文档**：https://next.vuex.vuejs.org/
- **用户端演示**：https://vvlog.igeekfan.cn (710277267@qq.com/123qwe)
