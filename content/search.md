---
title: "Search"
description: "Search for AI prompts by keywords, categories, tags, or professional roles."
layout: "page"
_build:
  render: "false"
---

<div class="search-container">
  <div class="search-box">
    <input type="text" id="search-input" placeholder="Search for prompts, categories, tags, or roles..." autocomplete="off">
    <div id="search-results"></div>
  </div>
</div>

<script src="https://cdn.jsdelivr.net/npm/fuse.js@6.4.6"></script>
<script>
  const searchInput = document.getElementById('search-input');
  const searchResults = document.getElementById('search-results');
  
  // 获取所有文章数据
  fetch('/index.json')
    .then(response => response.json())
    .then(data => {
      // 初始化Fuse.js
      const fuse = new Fuse(data, {
        keys: ['title', 'description', 'tags', 'categories', 'roles'],
        includeScore: true,
        minMatchCharLength: 2,
        threshold: 0.3
      });
      
      // 检查URL参数，支持按角色过滤
      const urlParams = new URLSearchParams(window.location.search);
      const roleFilter = urlParams.get('role');
      if (roleFilter) {
        searchInput.value = `role:"${roleFilter}"`;
        performSearch(fuse, `role:"${roleFilter}"`);
      }
      
      // 监听输入事件
      searchInput.addEventListener('input', (e) => {
        performSearch(fuse, e.target.value);
      });
    });
  
  function performSearch(fuse, query) {
    // 检查是否是角色搜索
    const roleMatch = query.match(/role:"([^"]+)"/);
    let results = [];
    
    if (roleMatch) {
      // 按角色过滤
      const role = roleMatch[1];
      results = fuse.getIndex().records.filter(record => {
        return record.v.roles && record.v.roles.includes(role);
      });
    } else if (query.length > 1) {
      // 普通搜索
      results = fuse.search(query);
    }
    
    displayResults(results, roleMatch ? true : false);
  }
  
  function displayResults(results, isRoleFilter = false) {
    if (results.length === 0 && searchInput.value.length > 1) {
      searchResults.innerHTML = '<p class="no-results">No results found.</p>';
      return;
    }
    
    let html = '<div class="results-list">';
    const items = isRoleFilter ? results : results.slice(0, 20);
    
    items.forEach(result => {
      const item = isRoleFilter ? result.v : result.item;
      html += `
        <div class="result-item">
          <h3><a href="${item.permalink}">${item.title}</a></h3>
          <p class="description">${item.description}</p>
          <div class="meta">
            <span class="date">${item.date}</span>
            ${item.categories ? item.categories.map(cat => `<span class="category">${cat}</span>`).join('') : ''}
            ${item.tags ? item.tags.map(tag => `<span class="tag">${tag}</span>`).join('') : ''}
            ${item.roles ? item.roles.map(role => `<span class="role">${role}</span>`).join('') : ''}
          </div>
        </div>
      `;
    });
    
    html += '</div>';
    searchResults.innerHTML = html;
  }
</script>

<style>
.search-container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
}

.search-box input {
  width: 100%;
  padding: 15px;
  font-size: 18px;
  border: 2px solid #ddd;
  border-radius: 8px;
  box-sizing: border-box;
}

.search-box input:focus {
  outline: none;
  border-color: #007bff;
}

.results-list {
  margin-top: 20px;
}

.result-item {
  padding: 20px;
  border-bottom: 1px solid #eee;
}

.result-item:last-child {
  border-bottom: none;
}

.result-item h3 {
  margin: 0 0 10px 0;
}

.result-item h3 a {
  color: #007bff;
  text-decoration: none;
}

.result-item h3 a:hover {
  text-decoration: underline;
}

.description {
  color: #666;
  margin-bottom: 10px;
}

.meta {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
}

.meta span {
  font-size: 12px;
  padding: 4px 8px;
  border-radius: 4px;
}

.date {
  background-color: #f8f9fa;
  color: #6c757d;
}

.category {
  background-color: #d4edda;
  color: #155724;
}

.tag {
  background-color: #cce7ff;
  color: #004085;
}

.role {
  background-color: #fff3cd;
  color: #856404;
}

.no-results {
  text-align: center;
  padding: 40px;
  color: #666;
}
</style>