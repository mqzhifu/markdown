任务：根据 Stitch 原型图生成静态前端页面（HTML + CSS）。

【资源目录】
D:\code\daily_record\static\html\richangjilu

该目录下包含：
1. doc.md（页面结构、设计描述）
2. 多张页面原型图（每张图对应一个页面）

-----------------------------------
【你的工作内容】

1. 根据目录中的每张原型图，逐一生成对应 HTML 页面。

2. 页面文件名规则：
- 原型图文件名如果是中文，请转换为英文文件名
- 使用小写字母 + 中划线命名
示例：
首页.png → home.html
个人中心.png → profile.html
订单详情.png → order-detail.html

3. 输出目录：
front/

4. 在 front 目录下生成：

front/
  *.html
  css/
    每个页面独立 css 文件
  icon/

5. CSS规范：
- 若 css 目录已存在则复用
- 每个页面一个独立 CSS 文件
- 文件名与 HTML 同名
示例：
home.html
css/home.css

6. 图标规范：
- 不使用 Google Icon
- 使用免费可商用图标（如 Lucide / Font Awesome Free / Heroicons）
- 所有图标资源存入：

front/icon/

7. 页面实现要求：
- 高度还原 Stitch 原型图布局、颜色、间距、字体层级
- 使用纯 HTML + CSS
- 暂不实现后端功能

8. 代码质量要求：
- 使用语义化 HTML
- 结构清晰，类名规范
- 避免重复 CSS
- 页面之间互不影响

----------------------------------
【执行方式】

先分析所有页面数量与命名，再按页面逐个生成。
每完成一个页面，告知：
页面名 + 已生成文件列表