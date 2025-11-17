# Directus CMS 集成改造计划

## 📋 项目概述

本文档详细说明将现有 Jekyll 实验室网站的数据管理从静态 YAML 文件迁移到 Directus CMS 的完整计划。

---

## 🔍 当前数据结构分析

### 1. 论文发表 (Citations) ✅ 已实现
**数据源**: `_data/citations.yaml`  
**状态**: 已通过 Directus 进行配置（参考实现）

**当前数据结构**:
```yaml
- id: doi:10.1371/journal.pcbi.1007128
  title: "论文标题"
  authors: ["作者1", "作者2"]
  publisher: "出版商"
  date: "2020-12-04"
  link: "https://doi.org/..."
  type: "paper"
  description: "描述内容"
  image: "图片URL"
  buttons:
    - type: "manubot"
      link: "链接"
    - type: "source"
      text: "按钮文本"
      link: "链接"
  tags: ["标签1", "标签2"]
  repo: "github仓库"
```

### 2. 项目 (Projects) 🔄 待迁移
**数据源**: `_data/projects.yaml`

**当前数据结构**:
```yaml
- title: "项目标题"
  subtitle: "副标题"
  group: "featured"  # 分组标识
  image: "images/photo.jpg"
  link: "https://github.com/..."
  description: "项目描述（支持Markdown）"
  repo: "github仓库名"
  tags: ["标签1", "标签2"]
```

**使用场景**:
- `projects/index.md` - 展示项目列表
- 支持按 group 筛选（featured/普通）
- 支持标签筛选和搜索

### 3. 团队成员 (Team Members) 🔄 待迁移
**数据源**: `_members/` 目录（Markdown文件）

**当前数据结构**:
```yaml
---
name: "成员姓名"
image: "images/photo.jpg"
role: "phd" | "postdoc" | "principal-investigator" | "programmer" | "undergrad" | "mascot"
group: "alum" | null  # 是否为校友
affiliation: "所属机构"
aliases: ["别名1", "别名2"]  # 可选
description: "简短描述"  # 可选
links:
  github: "用户名"
  orcid: "0000-0001-xxxx"
  email: "邮箱"
  twitter: "推特用户名"
  home-page: "个人主页"
---

成员详细介绍（Markdown格式的正文内容）
```

**使用场景**:
- `team/index.md` - 团队页面
- 按角色筛选展示（PI单独展示，其他成员一起展示）
- 每个成员有独立详情页

### 4. 博客文章 (Blog Posts) 🔄 待迁移
**数据源**: `_posts/` 目录（Markdown文件）

**当前数据结构**:
```yaml
---
title: "文章标题"
author: "作者ID"  # 对应 _members 中的文件名
tags: ["标签1", "标签2", "标签3"]
---

文章正文内容（Markdown格式）
```

**使用场景**:
- `blog/index.md` - 博客列表页
- 按时间排序
- 按标签筛选
- 支持搜索功能
- 每篇文章有独立详情页

### 5. 其他数据文件

#### 类型定义 (Types) ⚠️ 建议保留为静态
**数据源**: `_data/types.yaml`  
**说明**: 定义图标、链接模板等前端配置，建议保留为静态配置

#### 网站配置 (Config) ⚠️ 保留为静态
**数据源**: `_config.yaml`  
**说明**: Jekyll 配置文件，应保留为静态配置

#### ORCID配置 ⚠️ 保留为静态
**数据源**: `_data/orcid.yaml`  
**说明**: ORCID API配置，保留为静态

---

## 🗄️ Directus 集合设计方案

### 集合1: `publications`（论文发表）✅ 已实现
参考已实现的配置进行其他集合的设计。

**字段设计**:
| 字段名 | 类型 | 说明 | 是否必填 | 中英文支持 |
|--------|------|------|----------|------------|
| id | String (Primary) | 唯一标识符（如DOI） | 是 | - |
| title | String | 论文标题 | 是 | 双语 |
| title_en | String | 英文标题 | 否 | - |
| title_zh | String | 中文标题 | 否 | - |
| authors | JSON/Array | 作者列表 | 是 | - |
| publisher | String | 出版商 | 否 | 双语 |
| publisher_en | String | 英文出版商 | 否 | - |
| publisher_zh | String | 中文出版商 | 否 | - |
| date | Date | 发布日期 | 是 | - |
| link | String (URL) | 论文链接 | 否 | - |
| type | String | 类型（paper/book等） | 否 | - |
| description | Text (Markdown) | 描述 | 否 | 双语 |
| description_en | Text | 英文描述 | 否 | - |
| description_zh | Text | 中文描述 | 否 | - |
| image | Image | 论文配图 | 否 | - |
| buttons | JSON | 按钮配置 | 否 | - |
| tags | Many-to-Many (tags) | 标签 | 否 | - |
| repo | String | GitHub仓库 | 否 | - |
| status | String | 发布状态 | 是 | - |
| sort | Integer | 排序 | 否 | - |

### 集合2: `projects`（项目）🆕 新建

**字段设计**:
| 字段名 | 类型 | 说明 | 是否必填 | 中英文支持 |
|--------|------|------|----------|------------|
| id | UUID (Primary) | 自动生成 | 是 | - |
| title | String | 项目标题 | 是 | 双语 |
| title_en | String | 英文标题 | 是 | - |
| title_zh | String | 中文标题 | 否 | - |
| subtitle | String | 副标题 | 否 | 双语 |
| subtitle_en | String | 英文副标题 | 否 | - |
| subtitle_zh | String | 中文副标题 | 否 | - |
| group | String | 分组（featured/normal） | 否 | - |
| image | Image | 项目图片 | 否 | - |
| link | String (URL) | 项目链接 | 否 | - |
| description | Text (Markdown) | 项目描述 | 否 | 双语 |
| description_en | Text | 英文描述 | 是 | - |
| description_zh | Text | 中文描述 | 否 | - |
| repo | String | GitHub仓库 | 否 | - |
| tags | Many-to-Many (tags) | 标签 | 否 | - |
| status | String | 发布状态 (draft/published) | 是 | - |
| sort | Integer | 排序权重 | 否 | - |
| date_created | Timestamp | 创建时间 | 自动 | - |
| date_updated | Timestamp | 更新时间 | 自动 | - |

**索引**:
- `status` + `sort` (组合索引)
- `group` (单字段索引)

### 集合3: `team_members`（团队成员）🆕 新建

**字段设计**:
| 字段名 | 类型 | 说明 | 是否必填 | 中英文支持 |
|--------|------|------|----------|------------|
| id | UUID (Primary) | 自动生成 | 是 | - |
| slug | String (Unique) | URL友好标识 | 是 | - |
| name | String | 姓名 | 是 | 双语 |
| name_en | String | 英文姓名 | 是 | - |
| name_zh | String | 中文姓名 | 否 | - |
| image | Image | 照片 | 否 | - |
| role | String | 角色类型 | 是 | - |
| group | String | 分组（alum/current） | 否 | - |
| affiliation | String | 所属机构 | 否 | 双语 |
| affiliation_en | String | 英文机构 | 否 | - |
| affiliation_zh | String | 中文机构 | 否 | - |
| description | String | 简短描述 | 否 | 双语 |
| description_en | String | 英文简短描述 | 否 | - |
| description_zh | String | 中文简短描述 | 否 | - |
| bio | Text (Markdown) | 详细介绍 | 否 | 双语 |
| bio_en | Text | 英文详细介绍 | 否 | - |
| bio_zh | Text | 中文详细介绍 | 否 | - |
| aliases | JSON | 别名列表 | 否 | - |
| links | JSON | 社交链接 | 否 | - |
| status | String | 发布状态 | 是 | - |
| sort | Integer | 排序 | 否 | - |
| date_created | Timestamp | 创建时间 | 自动 | - |
| date_updated | Timestamp | 更新时间 | 自动 | - |

**links字段JSON结构**:
```json
{
  "github": "username",
  "orcid": "0000-0001-xxxx",
  "email": "email@example.com",
  "twitter": "username",
  "home-page": "https://...",
  "google-scholar": "userid",
  "linkedin": "username"
}
```

**role枚举值**:
- `principal-investigator` (主要研究员)
- `postdoc` (博士后)
- `phd` (博士生)
- `undergrad` (本科生)
- `programmer` (程序员)
- `mascot` (吉祥物)

**索引**:
- `slug` (唯一索引)
- `role` (单字段索引)
- `status` + `sort` (组合索引)

### 集合4: `blog_posts`（博客文章）🆕 新建

**字段设计**:
| 字段名 | 类型 | 说明 | 是否必填 | 中英文支持 |
|--------|------|------|----------|------------|
| id | UUID (Primary) | 自动生成 | 是 | - |
| slug | String (Unique) | URL友好标识 | 是 | - |
| title | String | 文章标题 | 是 | 双语 |
| title_en | String | 英文标题 | 是 | - |
| title_zh | String | 中文标题 | 否 | - |
| author | Many-to-One (team_members) | 作者 | 否 | - |
| content | Text (Markdown) | 文章正文 | 是 | 双语 |
| content_en | Text | 英文正文 | 是 | - |
| content_zh | Text | 中文正文 | 否 | - |
| excerpt | Text | 摘要 | 否 | 双语 |
| excerpt_en | Text | 英文摘要 | 否 | - |
| excerpt_zh | Text | 中文摘要 | 否 | - |
| tags | Many-to-Many (tags) | 标签 | 否 | - |
| image | Image | 封面图片 | 否 | - |
| date_published | Date | 发布日期 | 是 | - |
| status | String | 发布状态 | 是 | - |
| sort | Integer | 排序 | 否 | - |
| date_created | Timestamp | 创建时间 | 自动 | - |
| date_updated | Timestamp | 更新时间 | 自动 | - |

**索引**:
- `slug` (唯一索引)
- `date_published` (降序索引)
- `status` (单字段索引)
- `author` (外键索引)

### 集合5: `tags`（标签）🆕 新建（共享集合）

**字段设计**:
| 字段名 | 类型 | 说明 | 是否必填 | 中英文支持 |
|--------|------|------|----------|------------|
| id | UUID (Primary) | 自动生成 | 是 | - |
| name | String (Unique) | 标签名称 | 是 | - |
| name_en | String | 英文名称 | 是 | - |
| name_zh | String | 中文名称 | 否 | - |
| slug | String (Unique) | URL友好标识 | 是 | - |
| color | String | 标签颜色（HEX） | 否 | - |
| description | Text | 标签描述 | 否 | 双语 |
| date_created | Timestamp | 创建时间 | 自动 | - |

**使用说明**: 此集合供多个集合共享使用（projects, publications, blog_posts）

---

## 🔄 数据迁移方案

### 阶段1: 准备工作

#### 1.1 Directus环境配置
```bash
# 1. 安装Directus（使用Docker推荐）
docker-compose up -d

# 2. 配置环境变量
# 设置数据库连接
# 设置管理员账号
# 配置CORS允许Jekyll开发环境访问

# 3. 配置中英文多语言
# 在Directus设置中启用Translations
# 设置默认语言：英文(en-US)
# 添加第二语言：中文(zh-CN)
```

#### 1.2 创建集合结构
1. 按照上述设计在Directus中创建所有集合
2. 配置字段类型、验证规则
3. 设置关系字段（Many-to-One, Many-to-Many）
4. 配置权限（公开读取，管理员编辑）

### 阶段2: 数据迁移脚本

#### 2.1 迁移Projects数据

**创建迁移脚本**: `_scripts/migrate-projects.js`

```javascript
// 伪代码示例
const yaml = require('js-yaml');
const fs = require('fs');
const axios = require('axios');

const DIRECTUS_URL = 'http://localhost:8055';
const DIRECTUS_TOKEN = 'your-admin-token';

async function migrateProjects() {
  // 1. 读取现有YAML数据
  const data = yaml.load(fs.readFileSync('_data/projects.yaml', 'utf8'));
  
  // 2. 转换数据格式
  const projects = data.map(project => ({
    title_en: project.title,
    title_zh: project.title, // 需要人工翻译
    subtitle_en: project.subtitle,
    group: project.group || 'normal',
    image: project.image,
    link: project.link,
    description_en: project.description,
    repo: project.repo,
    tags: project.tags, // 需要先处理标签
    status: 'published'
  }));
  
  // 3. 上传到Directus
  for (const project of projects) {
    await axios.post(
      `${DIRECTUS_URL}/items/projects`,
      project,
      { headers: { Authorization: `Bearer ${DIRECTUS_TOKEN}` }}
    );
  }
}
```

#### 2.2 迁移Team Members数据

**创建迁移脚本**: `_scripts/migrate-members.js`

```javascript
// 需要处理：
// 1. 读取_members/目录下所有.md文件
// 2. 解析Front Matter和Markdown内容
// 3. 提取slug（文件名）
// 4. 上传图片到Directus
// 5. 创建成员记录
```

#### 2.3 迁移Blog Posts数据

**创建迁移脚本**: `_scripts/migrate-posts.js`

```javascript
// 需要处理：
// 1. 读取_posts/目录下所有.md文件
// 2. 从文件名提取日期
// 3. 解析Front Matter和Markdown内容
// 4. 关联作者（通过author字段匹配team_members的slug）
// 5. 处理标签关系
// 6. 创建文章记录
```

### 阶段3: 前端数据获取改造

#### 3.1 创建Directus API客户端

**新建文件**: `_scripts/directus-client.js`

```javascript
// Directus API客户端封装
class DirectusClient {
  constructor(url) {
    this.url = url;
  }
  
  async getProjects(filter = {}) {
    const params = new URLSearchParams({
      filter: JSON.stringify(filter),
      sort: 'sort',
      fields: '*,tags.tags_id.*'
    });
    
    const response = await fetch(`${this.url}/items/projects?${params}`);
    return await response.json();
  }
  
  async getMembers(filter = {}) {
    const params = new URLSearchParams({
      filter: JSON.stringify(filter),
      sort: 'sort',
      fields: '*'
    });
    
    const response = await fetch(`${this.url}/items/team_members?${params}`);
    return await response.json();
  }
  
  async getPosts(filter = {}) {
    const params = new URLSearchParams({
      filter: JSON.stringify(filter),
      sort: '-date_published',
      fields: '*,author.*,tags.tags_id.*'
    });
    
    const response = await fetch(`${this.url}/items/blog_posts?${params}`);
    return await response.json();
  }
  
  // 获取单个项目
  async getProject(slug) {
    const filter = { slug: { _eq: slug } };
    const result = await this.getProjects(filter);
    return result.data[0];
  }
  
  // 获取单个成员
  async getMember(slug) {
    const filter = { slug: { _eq: slug } };
    const result = await this.getMembers(filter);
    return result.data[0];
  }
  
  // 获取单篇文章
  async getPost(slug) {
    const filter = { slug: { _eq: slug } };
    const result = await this.getPosts(filter);
    return result.data[0];
  }
}

// 导出单例
window.directusClient = new DirectusClient('http://your-directus-url:8055');
```

#### 3.2 修改页面数据加载逻辑

**方案A: 构建时生成（推荐用于静态部署）**

创建Jekyll插件: `_plugins/directus_generator.rb`

```ruby
# 在Jekyll构建时从Directus获取数据
# 生成静态页面，保持现有模板结构
require 'net/http'
require 'json'

module Jekyll
  class DirectusGenerator < Generator
    safe true
    priority :highest

    def generate(site)
      # 获取Projects数据
      projects = fetch_directus('projects')
      site.data['projects'] = projects
      
      # 获取Members数据并生成页面
      members = fetch_directus('team_members')
      members.each do |member|
        site.pages << MemberPage.new(site, member)
      end
      
      # 获取Posts数据并生成页面
      posts = fetch_directus('blog_posts')
      posts.each do |post|
        site.posts.docs << PostDocument.new(site, post)
      end
    end
    
    def fetch_directus(collection)
      # API调用逻辑
    end
  end
end
```

**方案B: 客户端动态加载（用于需要实时更新的场景）**

修改相关include文件，在页面加载后通过JavaScript获取数据：

```javascript
// 在页面加载完成后
window.addEventListener('load', async () => {
  // 加载项目数据
  const projects = await window.directusClient.getProjects({
    status: { _eq: 'published' }
  });
  
  // 渲染项目列表
  renderProjects(projects.data);
});
```

---

## 📝 前端代码修改清单

### 1. Projects相关修改

#### 文件: `_data/projects.yaml`
**修改类型**: 🔄 改为从Directus获取或保留为备用

**方案1**: 删除此文件，完全使用Directus
**方案2**: 保留此文件作为fallback，在Directus不可用时使用

#### 文件: `projects/index.md`
**修改类型**: 🔄 可能需要调整数据字段映射

**需要修改的部分**:
```liquid
{# 修改前 #}
{% include list.html component="card" data="projects" filter="group == 'featured'" %}

{# 修改后 - 如果使用构建时生成，无需修改 #}
{# 如果使用客户端加载，需要添加容器和加载逻辑 #}
<div id="featured-projects" data-directus-collection="projects" data-filter="group:featured">
  {# Loading状态 #}
</div>
```

#### 文件: `_includes/card.html`
**修改类型**: ✅ 无需修改（字段映射保持兼容）

**需要确保的字段映射**:
- `title` → `title_en` 或 `title_zh`（根据语言）
- `subtitle` → `subtitle_en` 或 `subtitle_zh`
- 其他字段保持一致

### 2. Team Members相关修改

#### 目录: `_members/`
**修改类型**: ⚠️ 建议保留原文件作为备份

**处理方式**:
1. 迁移后将原文件移动到 `_members_backup/`
2. 或者在 `_config.yaml` 中调整collection配置

#### 文件: `team/index.md`
**修改类型**: 🔄 需要适配新的数据源

**修改点**:
```liquid
{# 修改前 #}
{% include list.html data="members" component="portrait" filter="role == 'pi'" %}

{# 如果使用Jekyll插件生成，data名称可能需要改为 team_members #}
{% include list.html data="team_members" component="portrait" filter="role == 'pi'" %}
```

#### 文件: `_includes/portrait.html`
**修改类型**: ✅ 检查字段映射

**需要映射的字段**:
- `name` → `name_en` / `name_zh`
- `description` → `description_en` / `description_zh`
- 其他字段保持一致

#### 文件: `_layouts/member.html`
**修改类型**: 🔄 需要适配bio字段

**修改内容**:
```liquid
{# 修改前 - 使用content #}
{{ content }}

{# 修改后 - 使用bio字段 #}
{{ page.bio_en | markdownify }}
{# 或根据当前语言选择 #}
```

### 3. Blog Posts相关修改

#### 目录: `_posts/`
**修改类型**: ⚠️ 建议保留原文件作为备份

#### 文件: `blog/index.md`
**修改类型**: 🔄 需要适配新的数据源

**修改点**:
```liquid
{# 修改前 #}
{% include list.html data="posts" component="post-excerpt" %}

{# 修改后 #}
{% include list.html data="blog_posts" component="post-excerpt" %}
```

#### 文件: `_includes/post-excerpt.html`
**修改类型**: 🔄 需要字段映射

**需要映射**:
- `title` → `title_en` / `title_zh`
- `excerpt` → `excerpt_en` / `excerpt_zh`
- `content` → `content_en` / `content_zh`
- `author` → 从关系字段获取作者对象

#### 文件: `_layouts/post.html`
**修改类型**: 🔄 适配新的内容字段

**修改内容**:
```liquid
{# 修改前 #}
{{ content }}

{# 修改后 #}
{{ page.content_en | markdownify }}
```

### 4. 通用修改

#### 文件: `_config.yaml`
**修改类型**: ➕ 添加Directus配置

**添加内容**:
```yaml
# Directus CMS配置
directus:
  url: "https://your-directus-instance.com"
  # 如果需要认证（公开读取可不配置token）
  # token: "your-access-token"
  
  # 配置集合映射
  collections:
    projects: "projects"
    members: "team_members"
    posts: "blog_posts"
    publications: "publications"
```

#### 新建文件: `_plugins/directus_generator.rb`
**修改类型**: ➕ 新建Jekyll插件

**功能**: 
- 在构建时从Directus获取数据
- 生成静态页面
- 缓存机制（避免每次构建都请求API）

#### 新建文件: `_plugins/directus_filters.rb`
**修改类型**: ➕ 新建Liquid过滤器

**功能**:
```ruby
# 多语言字段辅助过滤器
module Jekyll
  module DirectusFilters
    def localized_field(item, field_name, locale = 'en')
      locale = site.config['locale'] || locale
      localized_key = "#{field_name}_#{locale}"
      
      # 优先使用本地化字段，否则fallback到默认字段
      item[localized_key] || item[field_name] || ''
    end
  end
end

Liquid::Template.register_filter(Jekyll::DirectusFilters)
```

**使用示例**:
```liquid
{{ project | localized_field: 'title', 'zh' }}
```

---

## 🌐 多语言支持方案

### 方案1: URL路径区分（推荐）

**URL结构**:
- 英文: `/projects/`, `/team/`, `/blog/`
- 中文: `/zh/projects/`, `/zh/team/`, `/zh/blog/`

**实现方式**:
1. 在Directus中为每个集合的文本字段创建 `_en` 和 `_zh` 后缀版本
2. 在Jekyll中创建不同语言的页面目录
3. 通过URL参数或路径判断当前语言

**目录结构调整**:
```
/projects/index.md (英文)
/zh/projects/index.md (中文)
/team/index.md (英文)
/zh/team/index.md (中文)
```

### 方案2: 使用Directus Translations功能

**优势**:
- 统一的多语言管理
- 自动fallback机制
- 易于扩展更多语言

**实现**:
1. 在Directus中启用Translations
2. 配置语言（en-US, zh-CN）
3. API请求时通过 `lang` 参数获取对应语言

**API调用示例**:
```javascript
// 获取中文数据
fetch('http://directus/items/projects?lang=zh-CN')

// 获取英文数据
fetch('http://directus/items/projects?lang=en-US')
```

### 方案3: 混合方案（实用推荐）

**策略**:
- 短文本（标题、描述）：使用字段后缀（`_en`, `_zh`）
- 长文本（文章内容、Bio）：使用Directus Translations
- 前端根据当前语言智能选择字段

**优势**:
- 编辑界面简洁
- 性能较好（减少API请求）
- 灵活性高

---

## 🔒 权限和安全配置

### Directus权限设置

#### 1. 公开角色 (Public Role)
**配置用途**: 允许前端网站访问数据

**权限设置**:
- `projects`: 读取（Read） - 仅published状态
- `team_members`: 读取（Read） - 仅published状态
- `blog_posts`: 读取（Read） - 仅published状态
- `publications`: 读取（Read） - 仅published状态
- `tags`: 读取（Read）

**字段级权限**:
- 隐藏内部字段（如：sort, status）
- 暴露所有显示字段

#### 2. 管理员角色 (Administrator)
**权限**: 完全控制所有集合

#### 3. 编辑角色 (Editor) - 可选
**权限**: 
- 创建、编辑、删除内容
- 不能修改结构和配置

### API安全配置

```env
# .env配置
# 允许的域名（CORS）
CORS_ENABLED=true
CORS_ORIGIN=https://your-website.com,http://localhost:4000

# 限流配置
RATE_LIMITER_ENABLED=true
RATE_LIMITER_POINTS=50
RATE_LIMITER_DURATION=1

# 公开角色配置
PUBLIC_READ_ENABLED=true
```

---

## 🚀 实施步骤建议

### 第一阶段: 基础设施搭建（1-2天）
- [ ] 1.1 部署Directus实例
- [ ] 1.2 配置数据库和环境变量
- [ ] 1.3 创建所有集合结构
- [ ] 1.4 配置权限和访问控制
- [ ] 1.5 测试API访问

### 第二阶段: Projects迁移（2-3天）
- [ ] 2.1 在Directus中创建 `projects` 集合
- [ ] 2.2 创建和测试数据迁移脚本
- [ ] 2.3 执行数据迁移（包括图片上传）
- [ ] 2.4 创建Jekyll插件或修改前端代码
- [ ] 2.5 测试项目页面展示
- [ ] 2.6 添加中文翻译数据

### 第三阶段: Team Members迁移（2-3天）
- [ ] 3.1 在Directus中创建 `team_members` 集合
- [ ] 3.2 迁移成员数据和图片
- [ ] 3.3 修改团队页面模板
- [ ] 3.4 测试成员列表和详情页
- [ ] 3.5 添加中文翻译数据
- [ ] 3.6 验证社交链接正常工作

### 第四阶段: Blog Posts迁移（2-3天）
- [ ] 4.1 在Directus中创建 `blog_posts` 集合
- [ ] 4.2 迁移博客文章数据
- [ ] 4.3 建立作者关联关系
- [ ] 4.4 修改博客页面模板
- [ ] 4.5 测试文章列表和详情页
- [ ] 4.6 测试标签筛选功能
- [ ] 4.7 添加中文翻译（如需要）

### 第五阶段: 集成测试和优化（2-3天）
- [ ] 5.1 完整的端到端测试
- [ ] 5.2 性能优化（缓存、CDN）
- [ ] 5.3 SEO检查（确保meta标签正确）
- [ ] 5.4 响应式布局验证
- [ ] 5.5 浏览器兼容性测试
- [ ] 5.6 编写管理员使用文档

### 第六阶段: 上线和文档（1天）
- [ ] 6.1 备份原始数据
- [ ] 6.2 生产环境部署
- [ ] 6.3 DNS和HTTPS配置
- [ ] 6.4 监控和日志配置
- [ ] 6.5 编写内容编辑指南
- [ ] 6.6 团队培训

---

## 📊 数据备份和回滚策略

### 备份策略

#### 1. 原始数据备份
```bash
# 备份所有原始YAML和Markdown文件
mkdir -p _backup/$(date +%Y%m%d)
cp -r _data _backup/$(date +%Y%m%d)/
cp -r _members _backup/$(date +%Y%m%d)/
cp -r _posts _backup/$(date +%Y%m%d)/
```

#### 2. Directus数据备份
- 使用Directus的导出功能定期备份
- 配置数据库自动备份（每日）
- 备份上传的图片资源

#### 3. Git版本控制
- 所有代码修改通过Git管理
- 创建专门的分支进行迁移开发
- 每个阶段完成后打tag

### 回滚计划

**如果迁移失败**:
1. 恢复备份的原始文件
2. 切换回迁移前的Git分支
3. 停用Directus集成
4. 网站恢复到迁移前状态

**渐进式迁移建议**:
- 不要一次性删除所有原始文件
- 保持双轨运行一段时间
- 验证无误后再清理旧文件

---

## 🎯 预期效果和优势

### 对内容编辑者的优势
1. ✅ **可视化编辑界面** - 无需了解YAML语法或Markdown
2. ✅ **富文本编辑器** - 所见即所得的内容编辑
3. ✅ **图片管理** - 直接上传和管理图片，自动优化
4. ✅ **协作管理** - 多人同时编辑，权限控制
5. ✅ **草稿功能** - 可以保存草稿，预览后发布
6. ✅ **版本历史** - 自动保存修订历史，可以回滚

### 对开发者的优势
1. ✅ **API驱动** - RESTful API，易于集成
2. ✅ **解耦架构** - 内容和展示分离
3. ✅ **类型安全** - 字段验证和类型检查
4. ✅ **扩展性好** - 易于添加新字段和集合
5. ✅ **现代技术栈** - 支持GraphQL、Webhooks等

### 对网站管理的优势
1. ✅ **实时更新** - 内容修改后立即生效（如果使用客户端加载）
2. ✅ **数据安全** - 数据库备份，不担心文件丢失
3. ✅ **搜索友好** - 结构化数据更利于SEO
4. ✅ **性能优化** - 可以配置CDN和缓存策略
5. ✅ **多语言支持** - 统一管理多语言内容

---

## ⚠️ 注意事项和潜在风险

### 技术风险

#### 1. API依赖
**风险**: 网站依赖Directus API，如果服务不可用影响网站
**缓解措施**:
- 使用构建时生成方式（Jekyll插件）
- 设置API缓存和fallback机制
- 保留原始数据文件作为备用

#### 2. 性能问题
**风险**: 多次API请求可能影响页面加载速度
**缓解措施**:
- 使用静态生成而非客户端实时请求
- 配置CDN加速
- 实现适当的缓存策略
- 使用GraphQL减少请求次数

#### 3. SEO影响
**风险**: 如果使用客户端渲染可能影响SEO
**缓解措施**:
- 优先使用服务端渲染（Jekyll构建时生成）
- 确保meta标签正确
- 生成sitemap.xml
- 配置结构化数据（JSON-LD）

### 操作风险

#### 1. 数据迁移错误
**风险**: 数据格式转换错误导致信息丢失
**缓解措施**:
- 充分测试迁移脚本
- 先在测试环境运行
- 逐项验证迁移结果
- 保留原始数据备份

#### 2. 学习曲线
**风险**: 团队需要学习Directus使用
**缓解措施**:
- 提供详细的使用文档
- 进行培训和演示
- 设置简洁直观的界面
- 提供技术支持

### 维护风险

#### 1. 额外的基础设施
**风险**: 需要维护额外的Directus服务器
**缓解措施**:
- 使用托管服务（Directus Cloud）
- 配置自动备份
- 设置监控和告警
- 文档化运维流程

#### 2. 版本更新
**风险**: Directus版本更新可能带来不兼容
**缓解措施**:
- 锁定版本，计划升级
- 在测试环境先验证
- 关注官方更新日志
- 维护API版本兼容性

---

## 📚 参考资源

### Directus官方文档
- [Directus官方网站](https://directus.io/)
- [Directus文档](https://docs.directus.io/)
- [API参考](https://docs.directus.io/reference/introduction.html)
- [SDK文档](https://docs.directus.io/guides/sdk/)

### Jekyll相关
- [Jekyll官方文档](https://jekyllrb.com/docs/)
- [Liquid模板语言](https://shopify.github.io/liquid/)
- [Jekyll插件开发](https://jekyllrb.com/docs/plugins/)

### 多语言实现
- [Jekyll多语言插件](https://github.com/kurtsson/jekyll-multiple-languages-plugin)
- [Directus翻译功能](https://docs.directus.io/guides/headless-cms/content-translations.html)

---

## 📞 技术支持和FAQ

### 常见问题

#### Q1: 迁移到Directus后，原来的Git工作流还能用吗？
A: 可以。内容通过Directus管理，但代码和模板仍然通过Git管理。可以配置Directus的Webhooks在内容更新时触发自动部署。

#### Q2: 如何处理图片？
A: Directus提供完善的图片管理和存储。可以配置本地存储、S3或其他对象存储。支持图片自动优化和缩略图生成。

#### Q3: 是否需要修改很多前端代码？
A: 如果使用Jekyll插件方式，前端模板几乎不需要修改，只需要调整字段名映射。如果使用客户端加载，需要添加JavaScript代码。

#### Q4: 如何保证数据安全？
A: 
- 配置权限控制（只读公开访问）
- 启用HTTPS
- 定期备份数据库
- 使用环境变量管理敏感信息

#### Q5: 迁移需要多长时间？
A: 根据数据量和团队熟悉程度，预计10-15个工作日完成全部迁移和测试。

#### Q6: 现有的Citations集成是如何实现的？
A: 根据您提到的已实现的"论文发表"功能，建议查看该部分的实现代码作为其他集合迁移的参考模板。

---

## 📝 附录

### 附录A: Directus API调用示例

```javascript
// 获取所有已发布的项目（带标签）
fetch('http://directus/items/projects?filter[status][_eq]=published&fields=*,tags.tags_id.*&sort=sort')
  .then(res => res.json())
  .then(data => console.log(data));

// 获取特定成员的详细信息
fetch('http://directus/items/team_members?filter[slug][_eq]=john-doe&fields=*')
  .then(res => res.json())
  .then(data => console.log(data));

// 获取最近的5篇博客文章
fetch('http://directus/items/blog_posts?filter[status][_eq]=published&sort=-date_published&limit=5&fields=*,author.*')
  .then(res => res.json())
  .then(data => console.log(data));

// 按标签筛选项目
fetch('http://directus/items/projects?filter[tags][tags_id][name][_in]=software,resource')
  .then(res => res.json())
  .then(data => console.log(data));
```

### 附录B: Jekyll插件示例代码片段

```ruby
# _plugins/directus_data.rb
require 'net/http'
require 'json'
require 'uri'

module Jekyll
  class DirectusDataGenerator < Generator
    safe true
    priority :highest

    def generate(site)
      directus_url = site.config['directus']['url']
      
      # 获取Projects
      projects = fetch_collection(directus_url, 'projects')
      site.data['projects'] = projects
      
      # 获取Team Members
      members = fetch_collection(directus_url, 'team_members')
      members.each do |member|
        site.collections['members'].docs << MemberDocument.new(site, member)
      end
      
      # 获取Blog Posts
      posts = fetch_collection(directus_url, 'blog_posts')
      posts.each do |post|
        site.posts.docs << PostDocument.new(site, post)
      end
    end
    
    private
    
    def fetch_collection(base_url, collection)
      uri = URI("#{base_url}/items/#{collection}?filter[status][_eq]=published")
      response = Net::HTTP.get(uri)
      data = JSON.parse(response)
      data['data'] || []
    rescue StandardError => e
      Jekyll.logger.error "Directus:", "Failed to fetch #{collection}: #{e.message}"
      []
    end
  end
  
  class MemberDocument < Document
    def initialize(site, data)
      @site = site
      @data = transform_member_data(data)
      @content = data['bio_en'] || data['bio'] || ''
      
      # 设置路径
      @path = File.join(site.source, '_members', "#{data['slug']}.md")
      @relative_path = "_members/#{data['slug']}.md"
      @extname = '.md'
      @basename = data['slug']
    end
    
    private
    
    def transform_member_data(data)
      {
        'name' => data['name_en'] || data['name'],
        'image' => data['image'],
        'role' => data['role'],
        'links' => data['links'] || {}
      }
    end
  end
  
  class PostDocument < Document
    def initialize(site, data)
      @site = site
      @data = transform_post_data(data)
      @content = data['content_en'] || data['content'] || ''
      
      # 从date_published提取日期并构建路径
      date = Date.parse(data['date_published'])
      filename = "#{date.strftime('%Y-%m-%d')}-#{data['slug']}.md"
      
      @path = File.join(site.source, '_posts', filename)
      @relative_path = "_posts/#{filename}"
      @extname = '.md'
      @basename = data['slug']
    end
    
    private
    
    def transform_post_data(data)
      {
        'title' => data['title_en'] || data['title'],
        'author' => data['author'] ? data['author']['slug'] : nil,
        'tags' => data['tags']&.map { |t| t['tags_id']['name'] } || [],
        'date' => data['date_published']
      }
    end
  end
end
```

### 附录C: 数据迁移脚本完整示例

```javascript
// migrate-to-directus.js
const yaml = require('js-yaml');
const fs = require('fs');
const path = require('path');
const axios = require('axios');
const matter = require('gray-matter');
const FormData = require('form-data');

const DIRECTUS_URL = process.env.DIRECTUS_URL || 'http://localhost:8055';
const DIRECTUS_TOKEN = process.env.DIRECTUS_TOKEN;

const client = axios.create({
  baseURL: DIRECTUS_URL,
  headers: {
    'Authorization': `Bearer ${DIRECTUS_TOKEN}`,
    'Content-Type': 'application/json'
  }
});

// 上传图片到Directus
async function uploadImage(imagePath) {
  if (imagePath.startsWith('http')) {
    // 如果是URL，直接返回
    return imagePath;
  }
  
  const formData = new FormData();
  formData.append('file', fs.createReadStream(imagePath));
  
  const response = await client.post('/files', formData, {
    headers: formData.getHeaders()
  });
  
  return response.data.data.id;
}

// 迁移Projects
async function migrateProjects() {
  console.log('Migrating projects...');
  
  const data = yaml.load(fs.readFileSync('_data/projects.yaml', 'utf8'));
  
  for (const project of data) {
    try {
      // 上传图片
      let imageId = null;
      if (project.image) {
        imageId = await uploadImage(project.image);
      }
      
      // 创建项目记录
      await client.post('/items/projects', {
        title_en: project.title,
        subtitle_en: project.subtitle,
        group: project.group || 'normal',
        image: imageId,
        link: project.link,
        description_en: project.description,
        repo: project.repo,
        status: 'published',
        date_created: new Date().toISOString()
      });
      
      console.log(`✓ Migrated project: ${project.title}`);
    } catch (error) {
      console.error(`✗ Failed to migrate project: ${project.title}`, error.message);
    }
  }
}

// 迁移Team Members
async function migrateMembers() {
  console.log('Migrating team members...');
  
  const membersDir = '_members';
  const files = fs.readdirSync(membersDir).filter(f => f.endsWith('.md'));
  
  for (const file of files) {
    try {
      const content = fs.readFileSync(path.join(membersDir, file), 'utf8');
      const { data: frontmatter, content: bio } = matter(content);
      
      const slug = path.basename(file, '.md');
      
      // 上传照片
      let imageId = null;
      if (frontmatter.image) {
        imageId = await uploadImage(frontmatter.image);
      }
      
      // 创建成员记录
      await client.post('/items/team_members', {
        slug: slug,
        name_en: frontmatter.name,
        image: imageId,
        role: frontmatter.role,
        group: frontmatter.group,
        affiliation_en: frontmatter.affiliation,
        description_en: frontmatter.description,
        bio_en: bio,
        aliases: frontmatter.aliases,
        links: frontmatter.links,
        status: 'published'
      });
      
      console.log(`✓ Migrated member: ${frontmatter.name}`);
    } catch (error) {
      console.error(`✗ Failed to migrate member: ${file}`, error.message);
    }
  }
}

// 迁移Blog Posts
async function migratePosts() {
  console.log('Migrating blog posts...');
  
  const postsDir = '_posts';
  const files = fs.readdirSync(postsDir).filter(f => f.endsWith('.md'));
  
  for (const file of files) {
    try {
      const content = fs.readFileSync(path.join(postsDir, file), 'utf8');
      const { data: frontmatter, content: postContent } = matter(content);
      
      // 从文件名提取日期
      const dateMatch = file.match(/^(\d{4}-\d{2}-\d{2})-(.+)\.md$/);
      const date = dateMatch[1];
      const slug = dateMatch[2];
      
      // 查找作者ID
      let authorId = null;
      if (frontmatter.author) {
        const authorSlug = frontmatter.author;
        const authorResponse = await client.get(`/items/team_members?filter[slug][_eq]=${authorSlug}`);
        if (authorResponse.data.data.length > 0) {
          authorId = authorResponse.data.data[0].id;
        }
      }
      
      // 创建文章记录
      await client.post('/items/blog_posts', {
        slug: slug,
        title_en: frontmatter.title,
        author: authorId,
        content_en: postContent,
        date_published: date,
        status: 'published'
      });
      
      console.log(`✓ Migrated post: ${frontmatter.title}`);
    } catch (error) {
      console.error(`✗ Failed to migrate post: ${file}`, error.message);
    }
  }
}

// 主函数
async function main() {
  console.log('Starting migration to Directus...\n');
  
  try {
    await migrateProjects();
    console.log('\n');
    
    await migrateMembers();
    console.log('\n');
    
    await migratePosts();
    console.log('\n');
    
    console.log('✓ Migration completed successfully!');
  } catch (error) {
    console.error('✗ Migration failed:', error.message);
    process.exit(1);
  }
}

main();
```

### 附录D: 使用说明文档模板

```markdown
# Directus内容管理系统使用指南

## 登录系统
访问: https://your-directus-url.com/admin
使用您的账号和密码登录

## 管理项目 (Projects)

### 创建新项目
1. 点击左侧菜单 "Projects"
2. 点击右上角 "+" 按钮
3. 填写必填字段：
   - Title (EN): 英文标题
   - Title (ZH): 中文标题（可选）
   - Description (EN): 项目描述
4. 上传项目图片
5. 设置Status为"Published"
6. 点击保存

### 编辑现有项目
1. 在列表中找到要编辑的项目
2. 点击进入编辑页面
3. 修改需要的字段
4. 点击保存

## 管理团队成员 (Team Members)

### 添加新成员
1. 点击左侧菜单 "Team Members"
2. 点击 "+" 创建新成员
3. 填写信息（姓名、角色、照片等）
4. 在Links字段添加社交链接（JSON格式）
5. 保存

### 编辑成员信息
同项目编辑流程

## 管理博客文章 (Blog Posts)

### 发布新文章
1. 点击左侧菜单 "Blog Posts"
2. 创建新文章
3. 选择作者（从Team Members中选择）
4. 使用Markdown编辑器编写文章
5. 添加标签
6. 设置发布日期
7. 保存并发布

## 标签管理
所有标签在 "Tags" 集合中统一管理，可以被多个内容类型引用。

## 常见问题
Q: 如何预览修改效果？
A: 保存后，网站会在构建时自动更新（约5-10分钟）。

Q: 如何恢复删除的内容？
A: 联系管理员，可以从备份中恢复。
```

---

## 结语

本改造计划提供了从静态文件到 Directus CMS 的完整迁移路径。建议采用渐进式迁移策略，先完成一个模块的迁移和测试，验证无误后再进行下一个模块。

整个迁移过程需要前后端协作，建议组建包含以下角色的团队：
- 后端开发（负责Directus配置和数据迁移）
- 前端开发（负责Jekyll集成和页面调整）
- 内容编辑（参与测试和反馈）
- 项目协调（负责进度跟踪和问题解决）

迁移完成后，网站将具有更强的内容管理能力、更好的协作体验，以及更灵活的扩展性。

---

**文档版本**: v1.0  
**创建日期**: 2025-11-17  
**最后更新**: 2025-11-17  
**作者**: AI Assistant  
**状态**: 待审核

---
