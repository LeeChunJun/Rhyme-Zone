# Rhyme's Zone

## 项目结构
```
rhyme-blog                    # 项目根目录
├─assets
│  └─css                      # 样式文件目录
│
├─bin
│  └─index.cjs                # 执行文件，用于构建静态站点
│
├─dist
│  ├─assets                   # 资源结果输出目录
│  ├─posts                    # 博客网页输出目录
│  ├─index.html               # 静态站点首页
│  └─rss.xml                  # rss文件
│
├─posts                       # 博客文章MD文件目录
│  ├─android
│  ├─cross-platform
│  ├─front-end
│  ├─ios
│  └─recsys
├─template
│   └─pages
│      ├─index.ejs            # 首页模板文件
│      ├─post.ejs             # 博客模板文件
│      └─rss.ejs              # rss模板文件
├─miscc.yaml                  # 构建配置文件
├─README.md                   # 项目说明文件
└─package.json                # 项目配置文件
```

## 如何运行
```
cd rhyme-blog
node ./bin/index.cjs
```
