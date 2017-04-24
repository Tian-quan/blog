

```
NexT主题
http://theme-next.iissnan.com/getting-started.html
git clone https://github.com/iissnan/hexo-theme-next themes/next
```

```
//clone下来后
//1.删除 .deploy_git .idea public.
//2.clone 主题 (git clone https://github.com/iissnan/hexo-theme-next themes/next)
//3.执行git初始化(npm install hexo-deployer-git --save)


//cd 到blog目录, 初始化git
$ npm install hexo-deployer-git --save

// 安装 搜索插件
$npm install hexo-generator-searchdb --save

//根据_config.yml配置,上传到github
$ hexo d

//编译
$ hexo clean
$ hexo g

//新建一篇文章
hexo new xxx
```
```
//日常调试
hexo clean && hexo g && hexo s -p 8000
```
