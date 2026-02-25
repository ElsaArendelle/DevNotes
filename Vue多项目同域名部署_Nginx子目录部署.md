# Vue多项目同域名部署 (Nginx子目录部署)

## 需求背景
主项目部署在根目录 `http://192.168.1.7/`
新项目部署在子目录 `http://192.168.1.7/app2` (以 `app2` 为例)

## 第一步：修改新项目的 Vue 源码配置
因为新项目不再部署在域名的根目录中，如果不修改代码，打包后静态资源(JS/CSS)的路径会错误，导致页面空白，并且路由也会失效。

### 1. 修改打包的基础路径 (`vue.config.js` 或 `vite.config.js`)
如果是 Vue CLI (`vue.config.js`):
```javascript
module.exports = {
  // 指定项目的基础路径为子目录
  publicPath: '/app2/',  
  
  // 你原有的其他配置...
}
```

如果是 Vite (`vite.config.js`):
```javascript
import { defineConfig } from 'vite'

export default defineConfig({
  base: '/app2/',
  // 你原有的其他配置...
})
```

### 2. 修改路由的基准路径 (`router/index.js`)

**如果是 Vue 3 (Vue Router 4):**
```javascript
import { createRouter, createWebHistory } from 'vue-router'

const router = createRouter({
  // 把基础路径传给 history
  history: createWebHistory('/app2/'), 
  routes: [ /* 你的路由规则 */ ]
})
```

**如果是 Vue 2 (Vue Router 3):**
```javascript
import VueRouter from 'vue-router'

const router = new VueRouter({
  mode: 'history',
  base: '/app2/',   // 添加 base 属性
  routes: [ /* 你的路由规则 */ ]
})
```

完成以上修改后，执行打包命令（如 `npm run build`），生成 `dist` 文件夹（假设路径为 `G:/暂存/新项目/dist`）。

## 第二步：修改 Nginx 配置文件

在 Nginx 的主配置文件（或对应的 server 块）中，新增一个 `location /app2` 配置。

```nginx
server {
    listen       80;
    server_name  _;

    # 【原先的主项目】保持不变
    location / {
        root   "G:/暂存/AutoSquare网站/frontend/dist";
        index  index.html index.htm;
        # 【重要】Vue Router history 模式需要这行！
        try_files $uri $uri/ /index.html; 
    }

    # ==========================
    # 【新增】第二个项目的配置
    # ==========================
    location /app2 {
        # 注意！！这里用 alias 而不是 root
        alias  "G:/暂存/新项目/dist";
        index  index.html index.htm;

        # 【重要】防止刷新 404，注意最后变成了 /app2/index.html
        try_files $uri $uri/ /app2/index.html;
    }

    # API接口代理（原有配置保持不变）
    location /api/ {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   "G:/暂存/AutoSquare网站/frontend/dist";
    }
}
```

### 注意细节
1. 新增加的项目目录使用 `alias` 而不是 `root`，这会将请求路径 `/app2` 映射到对应的真实物理路径。
2. `try_files` 配置的最后一部分需要加上前缀路径（即 `/app2/index.html`）。

## 第三步：重启 Nginx
保存配置文件后，重新加载或重启 Nginx 让配置生效：
```bash
nginx -s reload
```

现在访问 `http://192.168.1.7/app2` 即可正常预览新项目。
