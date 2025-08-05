/* eslint-disable vue/multi-word-component-names */
<template>
  <div class="blog-container" :class="{ 'sidebar-open': isSidebarOpen }">
    <!-- 侧边栏 -->
    <aside class="sidebar" :class="{ active: isSidebarOpen }" @click="handleSidebarClick">
      <!-- 作者信息区域 -->
      <div class="author-section">
        <div v-if="loadingAuthorInfo" class="loading-author">
          <div class="spinner-small"></div>
          <p>加载作者信息...</p>
        </div>
        <div v-else>
          <div class="author-avatar">
            <img :src="authorInfo.avatar" :alt="authorInfo.name" />
          </div>
          <h3 class="author-name">{{ authorInfo.name }}</h3>
          <p class="author-bio">{{ authorInfo.bio }}</p>
          <div class="author-links">
            <a v-for="link in authorInfo.links" :key="link.name" :href="link.url" target="_blank" class="author-link">
              {{ link.name }}
            </a>
          </div>
        </div>
      </div>
      
      <!-- 导航菜单 -->
      <nav class="navigation">
        <div class="nav-header">
          <h4>文章列表</h4>
          <button @click="loadArticlesFromGitHub" :disabled="loadingArticlesList" class="refresh-btn">
            {{ loadingArticlesList ? '加载中...' : '刷新' }}
          </button>
        </div>
        
        <div v-if="errorMessage" class="error-message">
          {{ errorMessage }}
        </div>
        
        <div v-if="loadingArticlesList" class="loading-articles">
          <div class="spinner-small"></div>
          <p>正在从GitHub加载文章...</p>
        </div>
        
        <ul v-else class="nav-list">
          <li 
            v-for="article in articles" 
            :key="article.id"
            :class="{ active: selectedArticle?.id === article.id }"
            @click="selectArticle(article)"
            class="nav-item"
          >
            <span class="article-title">{{ article.title }}</span>
            <span class="article-date">{{ formatDate(article.date) }}</span>
            <div v-if="article.tags.length > 0" class="article-tags-preview">
              <span v-for="tag in article.tags.slice(0, 2)" :key="tag" class="tag-small">{{ tag }}</span>
            </div>
          </li>
        </ul>
      </nav>
    </aside>

    <!-- 主内容区域 -->
    <main class="main-content">
      <div v-if="loading" class="loading">
        <div class="spinner"></div>
        <p>加载中...</p>
      </div>
      
      <div v-else-if="selectedArticle" class="article-content">
        <header class="article-header">
          <button class="mobile-menu-btn inline" @click="toggleSidebar">
            {{ isSidebarOpen ? '✕' : '☰' }}
          </button>
          <h1>{{ selectedArticle.title }}</h1>
          <div class="article-meta">
            <span class="article-date">{{ formatDate(selectedArticle.date) }}</span>
            <span class="article-tags">
              <span v-for="tag in selectedArticle.tags" :key="tag" class="tag">{{ tag }}</span>
            </span>
          </div>
        </header>
        <div class="markdown-content" v-html="renderedContent"></div>
      </div>
      
      <div v-else class="welcome">
        <button class="mobile-menu-btn inline" @click="toggleSidebar">
          {{ isSidebarOpen ? '✕' : '☰' }}
        </button>
        <div class="welcome-content">
          <h1>欢迎来到我的博客</h1>
          <p>请从左侧选择一篇文章开始阅读</p>
        </div>
      </div>
    </main>
  </div>
</template>

<script>
import { marked } from 'marked';
import axios from 'axios';
import '@/assets/styles/blog.css';

export default {
  name: 'BlogPage',
  data() {
    return {
      loading: false,
      loadingArticlesList: false,
      selectedArticle: null,
      renderedContent: '',
      articles: [],
      errorMessage: '',
      isSidebarOpen: false,
      
      // GitHub仓库配置
      githubConfig: {
        owner: 'WilliamKahn',  // GitHub用户名或组织名
        repo: 'blogs',  // 仓库名
        branch: 'main',       // 分支名
        articlesPath: ''  // 文章所在目录路径，如果在根目录则留空 ''
      },
      
      authorInfo: {},
      loadingAuthorInfo: false
    };
  },
  methods: {
    toggleSidebar() {
      this.isSidebarOpen = !this.isSidebarOpen;
    },
    
    handleSidebarClick(event) {
      // 在移动端点击导航项后关闭侧边栏
      if (window.innerWidth <= 768 && event.target.closest('.nav-item')) {
        this.isSidebarOpen = false;
      }
    },
    
    async loadAuthorInfoFromGitHub() {
      this.loadingAuthorInfo = true;
      
      try {
        const { owner } = this.githubConfig;
        const userApiUrl = `https://api.github.com/users/${owner}`;
        
        console.log('Fetching author info from:', userApiUrl);
        
        const response = await axios.get(userApiUrl);
        const userData = response.data;
        
        // 更新作者信息
        this.authorInfo = {
          name: userData.name || userData.login, // 如果没有设置name，使用username
          bio: userData.bio || '在GitHub上分享代码和知识',
          avatar: userData.avatar_url,
          links: [
            { name: 'GitHub', url: userData.html_url },
            ...(userData.blog ? [{ name: 'Website', url: userData.blog.startsWith('http') ? userData.blog : `https://${userData.blog}` }] : []),
            ...(userData.twitter_username ? [{ name: 'Twitter', url: `https://twitter.com/${userData.twitter_username}` }] : []),
            ...(userData.email ? [{ name: 'Email', url: `mailto:${userData.email}` }] : [])
          ]
        };
        
        console.log('Loaded author info:', this.authorInfo);
        
      } catch (error) {
        console.error('Error loading author info from GitHub:', error);
        // 如果获取失败，保持默认的作者信息
        console.log('Using fallback author info');
      } finally {
        this.loadingAuthorInfo = false;
      }
    },

    async loadArticlesFromGitHub() {
      this.loadingArticlesList = true;
      this.errorMessage = '';
      
      try {
        const { owner, repo, branch, articlesPath } = this.githubConfig;
        
        // 构建GitHub API URL获取目录内容
        const path = articlesPath ? `/${articlesPath}` : '';
        const apiUrl = `https://api.github.com/repos/${owner}/${repo}/contents${path}?ref=${branch}`;
        
        console.log('Fetching articles from:', apiUrl);
        
        const response = await axios.get(apiUrl);
        const files = response.data;
        
        // 过滤出Markdown文件
        const markdownFiles = files.filter(file => 
          file.type === 'file' && 
          (file.name.endsWith('.md') || file.name.endsWith('.markdown'))
        );
        
        if (markdownFiles.length === 0) {
          this.errorMessage = '在指定的GitHub仓库中没有找到Markdown文件';
          return;
        }
        
        // 转换为文章格式
        this.articles = markdownFiles.map((file, index) => {
          const title = this.extractTitleFromFileName(file.name);
          const date = this.getFileLastModified();
          
          return {
            id: index + 1,
            title: title,
            date: date,
            tags: this.extractTagsFromTitle(title),
            fileName: file.name,
            githubUrl: file.download_url, // 使用download_url直接获取文件内容
            githubPath: file.path
          };
        });
        
        // 按日期排序（最新的在前）
        this.articles.sort((a, b) => new Date(b.date) - new Date(a.date));
        
        console.log('Loaded articles:', this.articles);
        
        // 默认选择第一篇文章
        if (this.articles.length > 0) {
          this.selectArticle(this.articles[0]);
        }
        
      } catch (error) {
        console.error('Error loading articles from GitHub:', error);
        this.errorMessage = `加载GitHub文章失败: ${error.message}`;
      } finally {
        this.loadingArticlesList = false;
      }
    },
    
    getFileLastModified() {
      // 简单返回当前日期，避免过多API调用
      return new Date().toISOString().split('T')[0];
    },
    
    extractTitleFromFileName(fileName) {
      // 移除文件扩展名
      let title = fileName.replace(/\.(md|markdown)$/i, '');
      
      // 将连字符和下划线替换为空格
      title = title.replace(/[-_]/g, ' ');
      
      // 首字母大写
      title = title.replace(/\b\w/g, char => char.toUpperCase());
      
      return title;
    },
    
    extractTagsFromTitle(title) {
      // 简单的标签提取逻辑，基于标题关键词
      const tags = [];
      const keywords = {
        'Vue': ['Vue.js', '前端'],
        'React': ['React', '前端'],
        'JavaScript': ['JavaScript', '前端'],
        'TypeScript': ['TypeScript', '前端'],
        'Docker': ['Docker', 'DevOps'],
        'Node': ['Node.js', '后端'],
        'Python': ['Python', '后端'],
        'API': ['API', '后端'],
        'CSS': ['CSS', '前端'],
        'HTML': ['HTML', '前端']
      };
      
      Object.entries(keywords).forEach(([keyword, keywordTags]) => {
        if (title.toLowerCase().includes(keyword.toLowerCase())) {
          tags.push(...keywordTags);
        }
      });
      
      return [...new Set(tags)]; // 去重
    },

    async selectArticle(article) {
      this.selectedArticle = article;
      this.loading = true;
      
      // 在移动端选择文章后关闭侧边栏
      if (window.innerWidth <= 768) {
        this.isSidebarOpen = false;
      }
      
      try {
        // 从 GitHub 获取 Markdown 内容
        const response = await axios.get(article.githubUrl);
        const markdownContent = response.data;
        
        // 配置marked选项
        marked.setOptions({
          breaks: true,
          gfm: true
        });
        
        // 渲染 Markdown
        this.renderedContent = marked(markdownContent);
      } catch (error) {
        console.error('Error loading article:', error);
        // 如果无法从 GitHub 获取，使用示例内容
        this.renderedContent = marked(this.getSampleContent(article.title));
      } finally {
        this.loading = false;
      }
    },
    
    getSampleContent(title) {
      return `# ${title}

这是一篇示例文章。由于 GitHub 地址可能需要配置，这里显示的是示例内容。

## 介绍

这是文章的介绍部分，会详细说明文章的主要内容和目标。

## 主要内容

### 第一部分

这里是文章的第一部分内容，包含了详细的技术说明和代码示例。

\`\`\`javascript
// 示例代码
function example() {
  console.log('Hello, World!');
}
\`\`\`

### 第二部分

这里是文章的第二部分，继续深入探讨相关话题。

## 总结

文章的总结部分，回顾主要观点和要点。

---

*感谢阅读！如有问题欢迎在评论区讨论。*`;
    },
    
    formatDate(dateString) {
      const date = new Date(dateString);
      return date.toLocaleDateString('zh-CN', {
        year: 'numeric',
        month: 'long',
        day: 'numeric'
      });
    }
  },
  
  mounted() {
    // 启动时从GitHub加载作者信息和文章
    this.loadAuthorInfoFromGitHub();
    this.loadArticlesFromGitHub();
    
    // 监听窗口大小变化，在桌面端自动关闭侧边栏
    this.handleResize = () => {
      if (window.innerWidth > 768 && this.isSidebarOpen) {
        this.isSidebarOpen = false;
      }
    };
    
    window.addEventListener('resize', this.handleResize);
    
    // 点击遮罩层关闭侧边栏
    this.handleOverlayClick = (event) => {
      if (this.isSidebarOpen && !event.target.closest('.sidebar') && !event.target.closest('.mobile-menu-btn')) {
        this.isSidebarOpen = false;
      }
    };
    
    document.addEventListener('click', this.handleOverlayClick);
    
    // ESC键关闭侧边栏
    this.handleKeydown = (event) => {
      if (event.key === 'Escape' && this.isSidebarOpen) {
        this.isSidebarOpen = false;
      }
    };
    
    document.addEventListener('keydown', this.handleKeydown);
  },
  
  beforeUnmount() {
    if (this.handleResize) {
      window.removeEventListener('resize', this.handleResize);
    }
    if (this.handleOverlayClick) {
      document.removeEventListener('click', this.handleOverlayClick);
    }
    if (this.handleKeydown) {
      document.removeEventListener('keydown', this.handleKeydown);
    }
  }
};
</script>
