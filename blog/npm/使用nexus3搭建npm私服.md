---
title: 使用nexus3搭建npm私服
date: {{ date }}
author: huangkai
tags:
    - npm
---

nexus3 安装请点击 [这里](https://github.com/huankai/blog-resources/blob/master/blog/Docker/Docker_02_%E5%AE%89%E8%A3%85%20nexus3.md) 参考 



# 一、配置npm仓库 #
在配置 npm　仓库前，可以先为 npm　类型的包创建　Blob Store ,分离其它类型仓库(如 jar 包 ，docker 镜像等)的存储位置，
操作如下图所示：

 ![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/npm/npm_nexus_00.png)

使用管理员账号登录 nexus 后，分别创建如下三个 npm仓库：

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/npm/npm_nexus_01.png)


## 1.1、添加 npm-proxy仓库： ##
选择上图的 Create repository -> npm(proxy) 

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/npm/npm_nexus_02.png)

其它保持默认即可

## 1.2、添加 npm-hosted仓库： ##

选择上图的 Create repository -> npm(hosted) 

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/npm/npm_nexus_03.png)

其它保持默认即可

## 1.3、添加 npm-public仓库： ##

选择上图的 Create repository -> npm(group) 

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/npm/npm_nexus_04.png)

其它保持默认即可


# 二、配置node 仓库地址 #

配置之前需要安装 node与 npm 环境
## 2.1、查询本机默认仓库地址 ##

```
F:\worksplace\kevin\design-vue>npm config get registry
https://registry.npm.taobao.org/

F:\worksplace\kevin\design-vue>

```

## 2.2、修改本机默认仓库地址 ##

```
F:\worksplace\kevin\design-vue>npm config set registry http://182.61.40.18:8081/repository/npm-public/
F:\worksplace\kevin\design-vue>npm config get registry
http://182.61.40.18:8081/repository/npm-public/

F:\worksplace\kevin\design-vue>
```
也可以直接在用户 HOME 目录下修改 .npmrc 文件，配置 registry 的值为 nexus的地址

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/npm/npm_nexus_05.png)

## 2.3、测试下载 npm包　##

如下图所示，安装　grunt 包，查看打印日志信息，可以已从指定的 nexus 下载相应的包了。

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/npm/npm_nexus_06.png)

等待安装完成后，可以看到私服中已缓存了请求过的包，下次再访问时就可以直接从npm私服中取了。

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/npm/npm_nexus_07.png)

# 三、配置nexus 认证 #

## 3.1、添加权限：##

添加 npm Bearer Token Realm ，从左边移动到右边保存即可

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/npm/npm_nexus_08.png)

## 3.2、创建 npm　角色：##

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/npm/npm_nexus_09.png)

## 3.3、创建 npm 用户：##

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/npm/npm_nexus_10.png)

## 3.4、本机 npm 登陆 ##

如下图，需要填写账号、密码、邮箱，账号与密码就是上面创建的 npm　账号与密码

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/npm/npm_nexus_11.png)

## 3.5、发布到 nexus ##

发布到Nexus 的仓库类型为 `hosted`，所以这里需要再次登陆 hosted的仓库

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/npm/npm_nexus_12.png)


在 package.json 中配置发布地址:

```
"publishConfig": {
    "registry": "http://182.61.40.18:8081/repository/npm-hosted/"
  },
```

发布:

```
F:\worksplace\kevin\design-vue>npm publish
npm notice
npm notice package: design-vue@0.1.0
npm notice === Tarball Contents ===
npm notice 126B   .editorconfig
npm notice 37B    .env
npm notice 1.3kB  src/assets/css/base.css
npm notice 107B   .env.development
npm notice 422B   public/index.html
npm notice 4.3kB  public/favicon.ico
npm notice 19B    src/store/actions.js
npm notice 1.4kB  src/network/address.js
npm notice 860B   src/router/address.js
npm notice 308B   babel.config.js
npm notice 1.3kB  src/network/clientApp.js
npm notice 585B   src/router/clientApp.js
npm notice 195B   src/router/dashboard.js
npm notice 1.3kB  src/network/dict.js
npm notice 1.2kB  src/router/dict.js
npm notice 101B   src/util/dictConstant.js
npm notice 390B   src/network/files.js
npm notice 423B   src/router/files.js
npm notice 397B   src/util/fsConstant.js
npm notice 370B   src/store/getters.js
npm notice 253B   src/network/headerApp.js
npm notice 1.1kB  src/components/address/index.js
npm notice 1.9kB  src/router/index.js
npm notice 948B   src/store/index.js
npm notice 163B   src/router/login.js
npm notice 1.3kB  src/main.js
npm notice 1.3kB  src/network/menu.js
npm notice 738B   src/router/menu.js
npm notice 162B   src/util/message.js
npm notice 605B   src/router/mine.js
npm notice 299B   src/util/moments.js
npm notice 164B   src/store/mutations-types.js
npm notice 347B   src/store/mutations.js
npm notice 1.1kB  src/network/organization.js
npm notice 634B   src/router/organization.js
npm notice 3.1kB  src/util/pageQuery.js
npm notice 376B   src/util/pagination.js
npm notice 1.1kB  src/network/permission.js
npm notice 642B   src/router/permission.js
npm notice 3.3kB  src/network/request.js
npm notice 978B   src/network/role.js
npm notice 575B   src/router/role.js
npm notice 1.4kB  src/network/schedule.js
npm notice 977B   src/router/schedule.js
npm notice 867B   src/util/stringUtils.js
npm notice 1.6kB  src/network/user.js
npm notice 559B   src/router/user.js
npm notice 3.0kB  src/util/validate.js
npm notice 169B   src/router/video.js
npm notice 2.1kB  vue.config.js
npm notice 4.4kB  public/data/address.json
npm notice 1.2kB  public/data/files.json
npm notice 431B   public/data/headerApp.json
npm notice 2.2kB  public/data/menu.json
npm notice 1.5kB  package.json
npm notice 363B   README.md
npm notice 43.5MB public/video/1.mp4
npm notice 1.2MB  src/assets/image/login_bg.png
npm notice 6.1kB  src/assets/image/logo.png
npm notice 19.4kB src/assets/logo.png
npm notice 32.5kB src/assets/image/user.png
npm notice 107B   .env.production
npm notice 6.4kB  src/views/emi/address/Address.vue
npm notice 1.6kB  src/components/cascader/AddressCascader.vue
npm notice 6.8kB  src/views/emi/address/AddressChild.vue
npm notice 12.8kB src/views/emi/address/AddressEdit.vue
npm notice 1.1kB  src/App.vue
npm notice 2.9kB  src/views/pms/mine/Base.vue
npm notice 7.8kB  src/views/pms/app/ClientApp.vue
npm notice 11.9kB src/views/pms/app/ClientAppEdit.vue
npm notice 4.3kB  src/views/dashboard/Dashboard.vue
npm notice 3.5kB  src/components/search/DateSearch.vue
npm notice 4.9kB  src/views/emi/dict/Dict.vue
npm notice 5.9kB  src/views/emi/dict/DictChild.vue
npm notice 5.4kB  src/views/emi/dict/DictChildEdit.vue
npm notice 3.8kB  src/views/emi/dict/DictEdit.vue
npm notice 1.5kB  src/views/file/FileAdd.vue
npm notice 6.2kB  src/views/file/Files.vue
npm notice 309B   src/components/layout/LayoutContent.vue
npm notice 3.8kB  src/components/layout/LayoutHeader.vue
npm notice 3.0kB  src/components/layout/LayoutSider.vue
npm notice 10.2kB src/views/login/Login.vue
npm notice 5.9kB  src/views/pms/menu/Menu.vue
npm notice 7.7kB  src/views/pms/menu/MenuEdit.vue
npm notice 9.5kB  src/views/pms/menu/MenuTreeList.vue
npm notice 2.5kB  src/views/pms/mine/Mine.vue
npm notice 6.1kB  src/views/pms/org/Organization.vue
npm notice 9.4kB  src/views/pms/org/OrganizationEdit.vue
npm notice 1.9kB  src/views/pms/org/OrganizationTree.vue
npm notice 6.8kB  src/views/pms/permission/Permission.vue
npm notice 4.3kB  src/views/pms/permission/PermissionEdit.vue
npm notice 5.8kB  src/views/pms/role/Role.vue
npm notice 5.9kB  src/views/pms/role/RoleEdit.vue
npm notice 7.9kB  src/views/schedule/Schedule.vue
npm notice 6.4kB  src/views/schedule/ScheduleEdit.vue
npm notice 5.4kB  src/views/schedule/ScheduleLog.vue
npm notice 5.9kB  src/views/pms/mine/Security.vue
npm notice 13.5kB src/views/pms/user/User.vue
npm notice 10.4kB src/views/pms/user/UserEdit.vue
npm notice === Tarball Details ===
npm notice name:          design-vue
npm notice version:       0.1.0
npm notice package size:  44.5 MB
npm notice unpacked size: 45.1 MB
npm notice shasum:        7e2ad8ea15fcb4b4e6dba0a85bb4ba920459856f
npm notice integrity:     sha512-ZkG1VIX5pylFJ[...]IXs+VkfzJov/w==
npm notice total files:   100
npm notice
+ design-vue@0.1.0

F:\worksplace\kevin\design-vue>

```

上面执行完成后，查看nexus 是否发布成功,如下图所示，｀design-vue｀已发布到成功

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/npm/npm_nexus_13.png)