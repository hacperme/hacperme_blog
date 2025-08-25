---
title: "搭建 Flask Htmlx daisyUI 开发环境"
date: 2023-10-28T20:39:01+08:00
lastmod: 2023-10-28T20:39:01+08:00
author: ["hacper"]
tags:
    - WEB
    - flask
    - Htmlx
    - Tailwind CSS
    - daisyUI
categories:
    - 笔记
description: "" # 文章描述，与搜索优化相关
summary: "使用 Flask Htmlx Tailwind CSS 和 daisyUI 搭建一个简单的 web 全栈开发环境。" # 文章简单描述，会展示在主页
# weight: # 输入1可以顶置文章，用来给文章展示排序，不填就默认按时间排序
slug: ""
draft: false # 是否为草稿
comments: true
showToc: true # 显示目录
TocOpen: true # 自动展开目录
autonumbering: true # 目录自动编号
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
searchHidden: false # 该页面可以被搜索到
showbreadcrumbs: true #顶部显示当前路径
# mermaid: true

---

## 目的

想掌握一套 web 全栈开发技术，用于开发、验证小项目，但不想花太大学习成本，需要一套轻量、简洁、易上手的技术栈。

查找一些资料之后，最终的选择如下

后端：flask
前端：htmlx+Tailwind CSS+daisyUI

- htmlx 可以在html中使用属性发送 AJAX 请求，处理服务器响应，可以替换大部分需要 JavaScript 来实现功能。
- Tailwind CSS 可以让我们在 html 中添加特定的工具类便能完成页面布局，无须单独写css。
- daisyUI 则是一个基于 Tailwind CSS 的组件库，简化 Tailwind CSS 的使用。

## 搭建步骤

安装 python 和 flask 环境

```bash
py -3 -m venv venv
.\venv\Scripts\Activate.ps1
```

```bash
pip install flask flask-assets
```

安装 Tailwind CSS 和 daisyUI

```bash
npm i -D daisyui@latest
npm install -D @tailwindcss/typography
```
如果不需要 daisyUI 组件，可以直接使用 Tailwind CSS 的命令行工具，而不需要安装 npm 和 nodejs。

```bash
pip install pytailwindcss  
```
如果要 daisyUI 暂时没办法，没有独立的安装工具。


下载 htmlx

下载 [htmx.min.js](https://unpkg.com/htmx.org/dist/htmx.min.js) 到 static\src\htmx.min.js 路径


##  整合 Tailwind CSS 到 flask

```bash
npx tailwindcss init
```

修改 tailwind.config.js 
```javascript
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    "./templates/**/*.html",
  ],
  theme: {
    extend: {},
  },
  plugins: [require("@tailwindcss/typography"), require("daisyui")],
  // daisyUI config (optional - here are the default values)
  daisyui: {
    themes: false, // true: all themes | false: only light + dark | array: specific themes like this ["light", "dark", "cupcake"]
    darkTheme: "dark", // name of one of the included themes for dark mode
    base: true, // applies background color and foreground color for root element by default
    styled: true, // include daisyUI colors and design decisions for all components
    utils: true, // adds responsive and modifier utility classes
    rtl: false, // rotate style direction from left-to-right to right-to-left. You also need to add dir="rtl" to your html tag and install `tailwindcss-flip` plugin for Tailwind CSS.
    prefix: "", // prefix for daisyUI classnames (components, modifiers and responsive class names. Not colors)
    logs: true, // Shows info about daisyUI version and used config in the console when building your CSS
  },
}
```

增加文件 static\src\main.css
```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```


然后编写 flask app 应用 app.py，使用 flask-assets 打包生成的 js 和 css文件

```python
from flask import Flask, render_template, request
from flask_assets import Environment, Bundle


app = Flask(__name__)

assets = Environment(app)
css = Bundle("src/main.css", output="dist/main.css")
js = Bundle("src/*.js", output="dist/main.js")

assets.register("css", css)
assets.register("js", js)

css.build()
js.build()


class User:
    id = 0
    def __init__(self, fname, lname, email ):
        User.id  += 1
        self.id = User.id 
        self.fname = fname
        self.lname = lname 
        self.email = email

    def search( self, word ):
        if (word is None):
            return False
        all = self.fname + self.lname + self.email
        return word.lower() in all.lower()

users = [
    User("abe", "vida", "abe@nowhere.com"),
    User("betty", "b", "bb@nowhere.com"),
    User("joe", "robinson", "jrobinson@nowhere.com"),

    User("Luis", "Cortes", "Luis@somewhere.com"),
    User("marty", "hinkle", "mhinkle@nowhere.com"),
    User("matthew", "robinson", "mrobinson@nowhere.com"),

    User("collin", "western", "cwest@nowhere.com"),
    User("marty", "hinkle II", "mhinkle2@nowhere.com"),
    User("joe", "robinson", "jrobinson@nowhere.com"),

    User("juan", "vida", "juanvida@nowhere.com"),
    User("marty", "hinkle III", "mhinkle3@nowhere.com"),
    User("zoe", "omega", "zoe@nowhere.com") 
]




@app.route("/")
def index():
    return render_template("index.html")

@app.route("/search", methods=["POST"])
def search():
    word = request.form.get("search")
    if (word is None or word == ""):
        return render_template("search.html", users=[])
    else:
        return render_template("search.html", users=filter(lambda u: u.search(word), users))
    
if __name__ == "__main__":
    app.run(debug=True)

```

编写模板文件

templates\base.html

```html
<!DOCTYPE html>
<html lang="en" class="h-full">

<head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewpoert" content="width=device-width, initial-scale=1.0">

    {% assets "css" %}
    <link rel="stylesheet" href="{{ ASSET_URL }}">
    {% endassets %}

    {% assets "js" %}
    <script type="text/javascript" src="{{ ASSET_URL }}"></script>
    {% endassets %}

    <title>Flask Htmlx daisyUI Demo </title>
</head>

<body class="h-full">
    {% block content %}
    {% endblock content %}
</body>

</html>

```

templates\index.html

```html
{% extends "base.html" %}

{% block content %}
<main class="max-w-screen-md mx-auto flex flex-col gap-8">
  <div class="w-full max-w-screen-md mx-auto">
    <header class="w-full flex items-center p-8" hx-boost="true">
      </h1>
      <a href="/" class="btn btn-secondary btn-outline">Home</a>
      </h1>
    </header>
  </div>
  <section>
    <div class="flex flex-col gap-8">
      <section class="hero bg-base-200 px-16 sm:px-32 py-24 sm:rounded-3xl">
        <div class="hero-content text-center">
          <div class="flex flex-col gap-6">
            <h1 class="text-4xl font-bold">
              Flask Htmlx daisyUI Demo
            </h1>
            <p class="text-lg">
              This is an simple demo of Flask and it's built in HTTP server app using Htmlx, TailwindCSS and DaisyUI.
            </p>
          </div>

        </div>
      </section>

      <section class="hero bg-base-200 px-16 sm:px-32 py-24 sm:rounded-3xl">
        <div class="hero-content text-left">
          <div class="flex flex-col gap-6">
            <h1 class="text-4xl font-bold">Search Contacts
              <span class="htmx-indicator">
                <img src="{{url_for('static', filename='img/bars.svg')}}" /> Searching...
              </span>
            </h1>
            <input class="form-control input input-bordered input-md w-full max-w-xs" type="text" name="search"
              placeholder="Begin Typing To Search Users..." hx-post="/search" hx-trigger="keyup changed delay:250ms"
              hx-target="#search-results" hx-indicator=".htmx-indicator">

            <table class="table table-xs">
              <thead>
                <tr>
                  <th>ID
                  </th>
                  <th>First
                    Name</th>
                  <th>Last
                    Name</th>
                  <th>Email
                  </th>
                </tr>
              </thead>

              <tbody id="search-results">
                {% include 'search.html' %}
              </tbody>
            </table>
          </div>
        </div>
      </section>

      <section class="hero bg-base-200 px-16 sm:px-32 py-24 sm:rounded-3xl">
        <div
          class="flex flex-col w-full max-w-md px-4 py-8 bg-white rounded-lg shadow dark:bg-gray-800 sm:px-6 md:px-8 lg:px-10">
          <div class="self-center mb-6 text-xl font-light text-gray-600 sm:text-2xl dark:text-white">
            Login To Your Account
          </div>
          <div class="mt-8">
            <form action="#" autoComplete="off">
              <div class="flex flex-col mb-2">
                <div class="flex relative ">
                  <span
                    class="rounded-l-md inline-flex  items-center px-3">
                    <svg width="15" height="15" fill="currentColor" viewBox="0 0 1792 1792"
                      xmlns="http://www.w3.org/2000/svg">
                      <path
                        d="M1792 710v794q0 66-47 113t-113 47h-1472q-66 0-113-47t-47-113v-794q44 49 101 87 362 246 497 345 57 42 92.5 65.5t94.5 48 110 24.5h2q51 0 110-24.5t94.5-48 92.5-65.5q170-123 498-345 57-39 100-87zm0-294q0 79-49 151t-122 123q-376 261-468 325-10 7-42.5 30.5t-54 38-52 32.5-57.5 27-50 9h-2q-23 0-50-9t-57.5-27-52-32.5-54-38-42.5-30.5q-91-64-262-182.5t-205-142.5q-62-42-117-115.5t-55-136.5q0-78 41.5-130t118.5-52h1472q65 0 112.5 47t47.5 113z">
                      </path>
                    </svg>
                  </span>
                  <input type="text" id="sign-in-email"
                    class="input input-bordered input-md w-full max-w-xs"
                    placeholder="Your email" />
                </div>
              </div>
              <div class="flex flex-col mb-6">
                <div class="flex relative ">
                  <span
                    class="rounded-l-md inline-flex  items-center px-3">
                    <svg width="15" height="15" fill="currentColor" viewBox="0 0 1792 1792"
                      xmlns="http://www.w3.org/2000/svg">
                      <path
                        d="M1376 768q40 0 68 28t28 68v576q0 40-28 68t-68 28h-960q-40 0-68-28t-28-68v-576q0-40 28-68t68-28h32v-320q0-185 131.5-316.5t316.5-131.5 316.5 131.5 131.5 316.5q0 26-19 45t-45 19h-64q-26 0-45-19t-19-45q0-106-75-181t-181-75-181 75-75 181v320h736z">
                      </path>
                    </svg>
                  </span>
                  <input type="password" id="sign-in-email"
                    class="input input-bordered input-md w-full max-w-xs"
                    placeholder="Your password" />
                </div>
              </div>
              <div class="flex items-center mb-6 -mt-4">
                <div class="flex ml-auto">
                  <a href="#"
                    class="link link-hover">
                    Forgot Your Password?
                  </a>
                </div>
              </div>
              <div class="flex w-full">
                <button type="submit" class="btn btn-secondary btn-outline">
                  Login
                </button>
              </div>
            </form>
          </div>
          <div class="flex items-center justify-center mt-6">
            <a href="#" target="_blank"
              class="link link-hover">
              <span class="ml-2">
                You don&#x27;t have an account?
              </span>
            </a>
          </div>
        </div>

      </section>
    </div>
  </section>
</main>
{% endblock content %}
```

templates\search.html

```html
{% if users %}
{% for user in users %}
<tr>
    <td>
        {{user.id}}</td>
    <td>
        {{user.fname}}</td>
    <td>
        {{user.lname}}</td>
    <td>
        {{user.email}}</td>
</tr>
{% endfor %}
{% endif %}
```

使用 Tailwind CSS 生成 CSS

```bash
npx tailwindcss -i ./static/src/main.css -o ./static/dist/main.css --minify
```

运行 app

```bash
(venv)$ python app.py
```

访问 http://127.0.0.1:5000/ 查看效果

![](https://github.com/hacperme/picx_hosting/raw/master/20210507/screencapture-127-0-0-1-5000-2023-10-30-01_44_51.3zjpb5uqib40.webp)

## 资料

- [Rapid Prototyping with Flask, htmx, and Tailwind CSS](https://testdriven.io/blog/flask-htmx-tailwind/)
- [htmx.org](https://htmx.org/)
- [flask](https://flask.palletsprojects.com/en/3.0.x/)
- [daisyui](https://daisyui.com/)