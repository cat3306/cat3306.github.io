# Hugo


## 介绍

hugo 是由go实现，一个快速建立静态网站的工具。详细教程可参考[hugo中文文档](https://www.gohugo.org/)
[正式官网](https://gohugo.io/)

## 安装

- 直接下载对应操作系统的二进制 [Hugo Releases](https://github.com/spf13/hugo/releases) ，放到`$GOPATH/bin/`

- [源码安装](https://github.com/gohugoio/hugo)

  ```bash
  cd $GOPATH/src/github.com
  git clone git@github.com:gohugoio/hugo.git
  go install
  ```

## hugo命令

```bash
# 生成站点
hugo new site ~/blog/Joker_null

# 在根目录(Joker_null)下执行，生成Markdown文件
hugo new posts/hugo_guide.md

# 在根目录(Joker_null)下执行,运行本地服务用于预览,-D参数是指草稿文件也加入预览文件中，localhost:1313
hugo server -D

# 打包生成静态网页，用于部署
hugo -d ./release

#详细命令参考
hugo help

#启用正式环境变量
hugo server --environment production
```

