# 1. Composer

## 常用命令

```text
composer update symfony/yaml --prefer-source  // 拉取源码（不可缓存）：
composer cleancache  // 删除缓存：
composer config -g repo.packagist composer https://packagist.laravel-china.org // 修改laravel源
composer uopdate "foo/bar:1.0.0" // 更新单个库：
composer dump-autoload --optimize  || composer dump-autoload -o 优化自动加载
```

## 以下是重点

1. 生产环境： composer dump-autoload --classmap-authoritative

> 在生产环境中使用使用权威类映射文件，这会让类映射文件中包含的所有类快速加载，而不必到磁盘文件系统进行任何检查。
>
> 此选项能使自动装填器始终快速返回。它还意味着如果由于某种原因在运行时生成类，则不允许自动加载。如果您的项目或任何依赖项执行此操作，那么您可能会在生产中遇到“未找到类”的问题\(此操作较少，对自己代码自信完全可以执行： -a \)
>
> 如果不自信，那至少执行： composer dump-autoload -o

1. 拉取源码： composer update symfony/yaml --prefer-source

> 这一般是因为墙的问题，例如github发布了一个版本，可以用这个快速获取最新版本

1. 加速安装： 使用 prestissimo 加速你的包安装

> composer global require hirak/prestissimo
>
> Composer 有个 hirak/prestissimo 插件，通过该插件能够以并行的方式进行下载，从而提高依赖包的安装速度。

1. 使用私有源

```javascript
    "repositories": [
        {
            "type": "git",
            "url": "git@git.***.com:exts/service-base.git"
        }
    ],
        "require": {
        "php": ">=7.1",
        "linghit-exts/service-base": "^2.0"
    },
```

1. require & require-dev

> require 所有环境都需要（prod） require-dev 一般是开发时需要库，一般为单元测试，文档生成的东西

1. 什么时候提交 composer.lock到版本库？

> 开发组件是不提交 （给其它组件、系统使用的） 开发项目时提交 （一个完整的web项目，脚本）

1. 发现composer 报错，怎么处理？

> 报错类型确定，是否具有权限 是否是镜像源的问题，尝试更换可用镜像源，并执行 composer update 如果是 composer install， 是否是composer.lock关联失效，尝试执行 composer update 尝试删除缓存： composer clean-cache 尝试 rm -rf vendor/

以上步骤基本可以解决绝大部分问题

## 这是精华，可以尝试阅读composer 源码

### composer dumpautoload -o 自动加载优化器

!&gt; 自动加载优化器

PSR-4 和 PSR-0 的自动加载规则在最终解析一个类之前需要去检查文件系统。这会导致自动加载的速度变得相当缓慢，但在开发环境中，这会是一种非常便捷的加载方式，因为当你新建了一个类时，加载器会立即发现并使用该类，而不需要你去重建自动加载器的配置。

这种加载规则导致的问题在生产环境中才真正体现出来，在生产环境中，你可以在每次部署之前非常轻松的重建配置，而且在部署之间不会随机出现新类，因此你不需要其一直检查文件系统，你通常都希望自动加载能尽可能快速的完成。

出于以上原因，Composer 提供了一些自动加载器的优化策略。

### 类映射生成

类映射生成实质上是将 PSR-4/PSR-0 规则转换为类映射规则。这使得一切都快很多，因为已知的类映射会立即返回路径，而 Composer 可以保证类在那里，_因此不需要文件系统检查_。

> php &gt; 5.6 中可以启用 opcache 类映射可以立即加载
>
> composer dumpautoload -o 一级类映射优化
>
> > 执行完 -o 之后可以查看下 autoload\_classmap.php 、autoload\_static.php，明显行数有所增加
>
> _composer --classmap-authoritative_ 2/A
>
> composer --apcu-autoloader 2/B APCu 缓存

