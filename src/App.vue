<script setup>
import { computed, ref, watchEffect, h, defineComponent } from 'vue';
import MarkdownIt from 'markdown-it';
import hljs from 'highlight.js/lib/core';
import javascript from 'highlight.js/lib/languages/javascript';
import typescript from 'highlight.js/lib/languages/typescript';
import json from 'highlight.js/lib/languages/json';
import bash from 'highlight.js/lib/languages/bash';
import shell from 'highlight.js/lib/languages/shell';
import markdownLang from 'highlight.js/lib/languages/markdown';

const search = ref('');
const activeSlug = ref('');
const funSubtitles = [
  'AI 垫刀，人类调味，代码和烤串一起冒烟',
  '键盘敲得像铁板烧，模型推得像爆炒蒜苔',
  '大模型当炉火，小模型当佐料，调试就是翻面',
  '提示词像葱姜蒜，切细了炒，才能香出 Bug 味',
  '深夜撸串写 Prompt，GPU 风扇当后厨排风'
];
const subtitle = ref(funSubtitles[Math.floor(Math.random() * funSubtitles.length)]);
hljs.registerLanguage('javascript', javascript);
hljs.registerLanguage('typescript', typescript);
hljs.registerLanguage('json', json);
hljs.registerLanguage('bash', bash);
hljs.registerLanguage('shell', shell);
hljs.registerLanguage('markdown', markdownLang);

const md = new MarkdownIt({
  html: true,
  linkify: true,
  highlight(str, lang) {
    if (lang && hljs.getLanguage(lang)) {
      try {
        return `<pre class="hljs"><code>${hljs.highlight(str, {
          language: lang,
          ignoreIllegals: true
        }).value}</code></pre>`;
      } catch (err) {
        console.warn('[highlight]', err);
      }
    }
    const escaped = MarkdownIt().utils.escapeHtml(str);
    return `<pre class="hljs"><code>${escaped}</code></pre>`;
  }
});

// 读取 posts 目录下的全部 Markdown 原始内容
const rawPosts = import.meta.glob('./posts/**/*.md', {
  eager: true,
  query: '?raw',
  import: 'default'
});

const postList = computed(() => {
  return Object.entries(rawPosts)
    .map(([path, content]) => {
      const slug = path.split('/').pop().replace(/\.md$/, '');
      const source = typeof content === 'string' ? content : content?.default || '';
      const titleMatch = source.match(/^# (.+)$/m);
      const title = titleMatch ? titleMatch[1] : slug;
      const relative = path.split('/posts/')[1] || '';
      const segments = relative.split('/').filter(Boolean);
      const categoryParts = segments.slice(0, -1);
      const categoryPath = categoryParts.length ? categoryParts : ['默认'];
      const categoryLabel = categoryPath.join(' / ');
      const html = md.render(source);
      return { slug, title, html, categoryParts: categoryPath, categoryLabel, path };
    })
    .sort((a, b) => {
      const catCompare = a.categoryLabel.localeCompare(b.categoryLabel, 'zh');
      return catCompare !== 0 ? catCompare : a.title.localeCompare(b.title, 'zh');
    });
});

const filteredPosts = computed(() => {
  const keyword = search.value.trim().toLowerCase();
  if (!keyword) return postList.value;
  return postList.value.filter(
    (post) =>
      post.title.toLowerCase().includes(keyword) ||
      post.categoryLabel.toLowerCase().includes(keyword) ||
      post.slug.toLowerCase().includes(keyword)
  );
});

const buildTree = (posts) => {
  const root = [];
  const nodeMap = new Map();

  posts.forEach((post) => {
    let currentLevel = root;
    post.categoryParts.forEach((part, idx) => {
      const key = post.categoryParts.slice(0, idx + 1).join('/');
      if (!nodeMap.has(key)) {
        const node = { label: part, key, children: [], isPost: false };
        nodeMap.set(key, node);
        currentLevel.push(node);
      }
      currentLevel = nodeMap.get(key).children;
    });
    currentLevel.push({ ...post, isPost: true });
  });

  const sortNodes = (nodes) => {
    nodes.sort((a, b) => {
      if (a.isPost !== b.isPost) return a.isPost ? 1 : -1;
      const aLabel = a.isPost ? a.title : a.label;
      const bLabel = b.isPost ? b.title : b.label;
      return aLabel.localeCompare(bLabel, 'zh');
    });
    nodes.forEach((n) => n.children && sortNodes(n.children));
  };

  sortNodes(root);
  return root;
};

const navTree = computed(() => buildTree(filteredPosts.value));

const currentPost = computed(() =>
  postList.value.find((post) => post.slug === activeSlug.value)
);

watchEffect(() => {
  if (!activeSlug.value && postList.value.length) {
    activeSlug.value = postList.value[0].slug;
  }
});

const handleSelect = (slug) => {
  activeSlug.value = slug;
};

const TreeNode = defineComponent({
  name: 'TreeNode',
  props: {
    node: {
      type: Object,
      required: true
    },
    activeSlug: {
      type: String,
      default: ''
    }
  },
  emits: ['select'],
  setup(props, { emit }) {
    const handle = (slug) => emit('select', slug);
    return () =>
      props.node.isPost
        ? h(
            'li',
            {
              class: ['nav-item', { active: props.node.slug === props.activeSlug }],
              onClick: () => handle(props.node.slug)
            },
            [h('span', props.node.title), h('span', { class: 'badge' }, props.node.categoryLabel)]
          )
        : h('li', { class: 'nav-group' }, [
            h('div', { class: 'nav-group__title' }, props.node.label),
            h(
              'ul',
              { class: 'nav-list nested' },
              props.node.children.map((child) =>
                h(TreeNode, {
                  node: child,
                  activeSlug: props.activeSlug,
                  onSelect: handle
                })
              )
            )
          ]);
  }
});
</script>

<template>
  <header class="hero">
    <div class="hero-text">
      <div class="title">曾记大排档</div>
      <div class="subtitle">{{ subtitle }}</div>
    </div>
  </header>

  <div class="shell">
    <aside class="panel">
      <input v-model="search" class="search" type="search" placeholder="搜索文章或分类..." />
      <ul class="nav-list">
        <TreeNode
          v-for="node in navTree"
          :key="node.isPost ? node.slug : node.key"
          :node="node"
          :activeSlug="activeSlug"
          @select="handleSelect"
        />
      </ul>
      <div v-if="!navTree.length" class="empty">暂无匹配的文章</div>
    </aside>

    <main class="panel content">
      <template v-if="currentPost">
        <div class="article-header">
          <h1>{{ currentPost.title }}</h1>
          <span class="article-meta">{{ currentPost.categoryLabel }}</span>
        </div>
        <div class="md-body" v-html="currentPost.html" />
      </template>
      <div v-else class="empty">当前没有可展示的 Markdown 文件</div>
    </main>
  </div>
</template>
