# 参考 https://jekyllrb.com/tutorials/using-jekyll-with-bundler/
bundle init -> 生成gemfile文件 

cd gemfile add
gem 'jekyll', '~> 4.4'
gem 'minima', '~> 2.5'
gem 'jekyll-seo-tag', '~> 2.8'
gem 'webrick'
gem "jekyll-paginate"

再次bundle install

bundle exec jekyll serve


如果你想在 **Jekyll** 博客中添加 **Gitalk**（基于 GitHub Issues 的评论系统），可以按照以下步骤进行配置。Gitalk 类似于 utterances，但提供更多的自定义选项。

---

## **1. 注册 GitHub OAuth App（获取 `clientID` 和 `clientSecret`）**
Gitalk 需要 GitHub OAuth 授权才能访问你的仓库评论（Issues）。  

### **步骤：**
1. 访问 [GitHub Developer Settings](https://github.com/settings/developers) → **OAuth Apps** → **New OAuth App**。  
2. 填写信息：
   - **Application name**: `Gitalk for My Blog`（可自定义）
   - **Homepage URL**: `https://yourblog.com`（你的博客地址）
   - **Authorization callback URL**: `https://yourblog.com`（和 Homepage 相同）
3. 点击 **Register application**，然后复制：
   - **Client ID** → 用于 `clientID`  
   - **Client Secret** → 用于 `clientSecret`  

---

## **2. 在 Jekyll 中配置 Gitalk**
### **方法 1：直接修改 `_config.yml`**
在 `_config.yml` 中添加：
```yaml
gitalk:
  enable: true
  clientID: "你的ClientID"          # 替换成你的 GitHub OAuth Client ID
  clientSecret: "你的ClientSecret"  # 替换成你的 GitHub OAuth Client Secret
  repo: "你的仓库名"               # 如 qiubaiying.github.io
  owner: "你的GitHub用户名"         # 如 qiubaiying
  admin: "你的GitHub用户名"         # 可以填多个，如 ["user1", "user2"]
  distractionFreeMode: true        # 是否启用无干扰模式（遮罩层）
  language: "zh-CN"                # 语言（可选 en, zh-CN, zh-TW 等）
```
> ⚠️ **注意：**  
> - `clientSecret` 是敏感信息，建议不要直接提交到公开仓库（可以用环境变量或 `_config_dev.yml` 代替）。  
> - 确保你的仓库 **已开启 Issues 功能**（Settings → Features → Issues）。

---

### **方法 2：创建 `_includes/gitalk.html`（推荐）**
1. 在 `_includes/` 目录下创建 `gitalk.html`：
   ```html
   <!-- _includes/gitalk.html -->
   <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/gitalk@1/dist/gitalk.css">
   <script src="https://cdn.jsdelivr.net/npm/gitalk@1/dist/gitalk.min.js"></script>
   
   <div id="gitalk-container"></div>
   <script>
     const gitalk = new Gitalk({
       clientID: "{{ site.gitalk.clientID }}",
       clientSecret: "{{ site.gitalk.clientSecret }}",
       repo: "{{ site.gitalk.repo }}",
       owner: "{{ site.gitalk.owner }}",
       admin: "{{ site.gitalk.admin }}",
       id: window.location.pathname,  // 确保每篇文章的评论独立
       distractionFreeMode: {{ site.gitalk.distractionFreeMode }},
       language: "{{ site.gitalk.language | default: 'en' }}"
     });
     gitalk.render("gitalk-container");
   </script>
   ```
2. 在 `_layouts/post.html` 中引入：
   ```html
   {% if site.gitalk.enable and page.comments != false %}
       {% include gitalk.html %}
   {% endif %}
   ```

---

## **3. 测试 Gitalk**
1. 启动 Jekyll 本地服务器：
   ```bash
   bundle exec jekyll serve
   ```
2. 访问博客文章，底部应该会出现 **Gitalk 评论框**。  
3. **首次加载** 需要登录 GitHub 授权，并自动创建 Issue（标题为当前页面路径）。  

---

## **4. 常见问题**
### **Q1: 评论框不显示？**
- 检查 `clientID` 和 `clientSecret` 是否正确。  
- 确保仓库是 **公开的**（Public）。  
- 检查浏览器控制台（F12 → Console）是否有错误。  

### **Q2: 如何管理评论？**
- 所有评论存储在仓库的 **Issues** 标签页。  
- 你可以手动关闭、删除或回复 Issue。  

### **Q3: 如何修改样式？**
在 `_sass/_custom.scss` 或 CSS 文件添加：
```css
#gitalk-container {
    max-width: 800px;
    margin: 2rem auto;
    padding: 1rem;
}
```

---

## **5. 总结**
| 步骤 | 操作 |
|------|------|
| **1. 注册 OAuth App** | [GitHub Developer Settings](https://github.com/settings/developers) |
| **2. 配置 `_config.yml`** | 填写 `clientID`, `clientSecret`, `repo` 等 |
| **3. 创建 `gitalk.html`** | 引入 JS/CSS 并初始化 Gitalk |
| **4. 引入到 `post.html`** | `{% include gitalk.html %}` |
| **5. 测试** | 访问博客，首次评论会自动创建 Issue |

现在你的 Jekyll 博客就集成了 **Gitalk** 评论系统！🎉 如果有问题，欢迎继续提问。