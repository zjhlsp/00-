### **Shopify Theme + TailWind优化**



#### **背景**

目前them项目开发自定义模块组件步骤如下

1. sections下创建liquid文件，构思好schema数据结构
2. snippets下创建小组件（如需要）
3. 编写snippets对应css文件
4. 编写sections组件css文件，引入snippets
5. shopify后台填写数据来源

单一个自定义组件下来便创建两个css文件，且有诸多问题：

1. 首先开发流程就需要手写样式文件相当繁琐，不同端通过媒体查询来进行适配，一个css文件要写两个端端样式
2. 增大项目体积影响网页性能，且assets目录下创建文件太多不好管理
3. 对于一些特殊需求（比如开学季、圣诞季变化）需要将页面组件等主题颜色更改，这样一来要么你在代码中将颜色设置为可配置，要么在主题变化时候一个一个组件等改，无论哪种方案开发成本都相当大



#### **目标**

其实步骤3，4中，开发常用样式大多雷同，可以引入全局原子化样式进行优化，一是不用再繁琐的定义类名，各个端的适配更加容易二是减少css文件变相优化了网站性能，一举两得

1. 简化开发流程
2. 缩小项目体积，优化性能
3. 组件主题样式可配置化



#### **TailWind原子化CSS方案**

原子化 CSS 的核心理念是将样式分解成非常小的、**粒度细致的**、单一功能的 CSS 类（低级工具类）。这些类通常只执行一个特定的任务，比如设置**间距、颜色、对齐、宽高**等。开发者可以通过组合这些类来构建复杂的用户界面，而不需要编写自定义的 CSS。

通过引入这些低级工具类，就可直接在liquid模版中实现样式更高，无需再另加样式，问题1、2迎刃而解，栗子如下

```html

<div class="bg-blue-500 text-white font-bold p-4 rounded-lg">
  Tailwind CSS Example
</div>
```

上面的代码中使用了多个低级工具类：

- `bg-blue-500`: 设置背景色为蓝色。
- `text-white`: 设置文字颜色为白色。
- `font-bold`: 设置字体为粗体。
- `p-4`: 设置 16px 的内边距。
- `rounded-lg`: 设置圆角。

Tailwind 强调**高度可配置**。可以通过修改配置文件 `tailwind.config.js` 来自定义颜色、间距、字体等，还可以通过 lg: max-lg: 等样式语法直接完成各端适配，最后设置**css变量**的方式实现主题修改，这样问题3也解决了



#### **思路**

首先要考虑的是我们无需引入tailwind中所有的样式，这样体积太大了，引入用到的样式即可。

其次中shopify theme 中的主题大多都会有一些默认的sections以及它的样式，还有一些默认的全局样式，因此要保证不会发生与tailwind的样式冲突

最后在代码开发过程中，要动态的更新tailwind中用到的样式

已知shopify中的所有样式文件都从assets目录中引入，因此我们最容易想到的方案就是在assets 添加tailwind的样式文件，并在layout中全局引入这个样式文件即可



#### **实现**

1. 安装tailwind包

   ```bash
   npm install -D tailwindcss
   ```

2. 根目录下创建tailwind.css，作为引入样式的源文件

   tailwind分为三个部分，base是基于全局的样式（body h1 h2 等）会与theme冲突，注意不要引入

   ```css
   /* @tailwind base; */
   @tailwind components;
   @tailwind utilities;
   ```

3. assets文件下创建tailbase.css (名称随意)，作为从tailwind引入到项目中的样式

   并在layout下的 theme.liquid 中全局引入

4. 创建tailwind.config.js 对tailwind进行预配置

   ```js
   /** @type {import('tailwindcss').Config} */
   
   module.exports = {
     prefix: 'tw-',
     content: [
       './**/*.liquid',   // 监视所有 .liquid 文件
       './**/*.css',       // 监视所有 .css 文件
       './**/*',
       '*'
     ],
     theme: {
       screens: {
         sm: '320px',
         md: '750px',
         lg: '990px',
         xlg: '1440px',
         x2lg: '1920px',
         pageMaxWidth: '1440px',
       },
       extend: {
         fontFamily: {
           heading: 'var(--font-heading-family)',
         },
       },
     },
     plugins: [],
   };
   ```

   这里主要做了以下工作

   1. 设置prefix前缀 预防与现有样式中冲突，后续使用工具类需要加对应前缀
   2. 设置屏幕模版，为端适配做准备
   3. 监听代码文件，用来代码联想和动态导入

5. 最后一步，安装concurrently库并配置package.json。npm run dev 启动项目。我的文件如下

   ```json
   {
     "scripts": {
       "dev": "concurrently  -c \"auto\" \"npm:dev:*\"",
       "dev:tailwind": "npx tailwindcss -i ./assets/tailwind.css -o ./assets/application.css --watch",
       "dev:shopify": "shopify theme dev"
     },
     "devDependencies": {
       "@shopify/prettier-plugin-liquid": "^1.5.0",
       "concurrently": "^8.2.2",
       "tailwindcss": "^3.4.10"
     }
   }
   
   ```



#### **效果**

下图是动态build并引入项目的css文件

<img src="/Users/zou/Desktop/截屏2024-09-07 17.44.11.png" alt="截屏2024-09-07 17.44.11" style="zoom:50%;" />

每次更改代码，重新生成样式文件

![截屏2024-09-07 17.47.16](/Users/zou/Desktop/截屏2024-09-07 17.47.16.png)

代码使用，可以看会自带联想，写样式很方便

![截屏2024-09-07 17.49.45](/Users/zou/Desktop/截屏2024-09-07 17.49.45.png)

![截屏2024-09-07 17.50.56](/Users/zou/Desktop/截屏2024-09-07 17.50.56.png)