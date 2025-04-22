---
date: 2025-04-22
authors: 
  - Jimmy
categories:
  - Tech
---
# 自建服务的简单记录

之前陆陆续续都部署了一些自建服务，感觉到现在已经不少，所以想要记录一下，方便日后的维护和迭代。这边主要记录一下一些使用的开源框架，配置选项，以及服务部署需要的一些注意的点。

> *碎碎念：自建服务虽然方便，但也麻烦（恰似没说hhhhhh，尤其对于我这种懒人来说，经常忘记，所以自动化维护非常重要了qwq*

<!-- more -->

首先，先简单列举一下目前已经部署的服务，部分链接出于安全性考量就隐藏了，~~防止被恶意打爆~~

!!! info "服务列表"
    - [Personal Homepage](https://jsta.top)
    - [Note](https://note.jsta.top)
    - [Temp Mail](https://mail.jsta.top)
    - [Moe Mail](https://mail.j2to.com)
    - [Lobe Chat](https://chat.j2to.com)
    - [Status Uptime](https://status.j2to.com)
    - [Bark](/#)
    - [Bitwarden](/#)

## Personal Homepage

这个没什么好说的其实，就是一个静态网页，部署在了国内服务器上，因此做了备案，作为一个简单的导航页绰绰有余了。但是为了安全起见还是套了一层cf，所以国内访问就见仁见智了（笑

## Note

就是本Note，部署在`Github Page`上，也属于静态页面，具体详情可见 [Note 搭建维护小记](/blog/2025/02/23/note-搭建维护小记/)

## Temp Mail

这是基于一个开源项目 [dreamhunter2333/cloudflare_temp_email](https://github.com/dreamhunter2333/cloudflare_temp_email/) ，功能相对完善，除了界面UI审美不咋地，其他都还不错。大部分邮箱功能都集成在一起了，已经不太属于轻量化的Temp Mail了。整体基于cf大善人提供的Worker&Page，所以不需要服务器，运行成本很低。

现在去看了一下，最近更新了很多新的feature，比如基于rust的wasm，基于 [zou-yu/worker-mailer](https://github.com/zou-yu/worker-mailer/) 的SMTP邮件发送服务更加方便了，之前还得依靠 [Resend](https://resend.com/) 的API来发送邮件，虽然免费额度也很高，但是还是有点麻烦的。现在直接集成在一起了，全部可以依靠cf管理（再次高呼cf大善人万岁！hhhhh

~~找个时间更新一下（逃~~

## Moe Mail

选择这个 [beilunyang/moemail](https://github.com/beilunyang/moemail) 主要是UI简单，让人赏心悦目，但是功能相对比较简陋，单纯作为临时收邮件倒也挺方便的。

![Homepage](https://camo.githubusercontent.com/9c2c104a2aed2787d7c1fc3cf42811caa790f9ce9084105ca9b702f6bfaf28b0/68747470733a2f2f7069632e6f74616b752e72656e2f32303234313230392f415141447773557847396b3175565a2d2e6a7067 "Homepage")
![Panel](https://camo.githubusercontent.com/981964990495c5144b2c3d00fa05a0d5b760f4229de73415808723b1c11bd7d8/68747470733a2f2f7069632e6f74616b752e72656e2f32303234313230392f415141447738557847396b3175565a2d2e6a7067 "Panel")
![Admin Center](https://camo.githubusercontent.com/411d04924435770d568c62d5292d9e776352ea8e1126f3b3e8f02048783946f9/68747470733a2f2f7069632e6f74616b752e72656e2f32303234313232372f415141445673497847374f7a6346642d2e6a7067 "Admin Center")

另外它也可以完全依赖cf部署，自用零成本运行。

## Lobe Chat

[lobehub/lobe-chat](https://github.com/lobehub/lobe-chat) 相对功能完善的 web 端 `chatbox`，尤其是自带服务器的版本，而且更新频率高，用起来非常顺手，文档相对很完善，部署起来也毫无压力。

目前我是依靠 [Vercel](https://vercel.com/) 部署的，有免费的 `Serveless Database` 可以使用，另外知识库依靠 cf 的 S3 存储，自用的话基本都是在免费额度内，运行成本很低，利用 `Github Action` 可以实现自动更新部署。有关用户认证部分我是使用了 [Clerk](https://clerk.com/) 的服务，但是相对比较庞大，其实用 `OAuth` 也不错，但是我懒得换了，将就用着吧 O(￣▽￣)ノ

## Status Uptime

[lyc8503/UptimeFlare](https://github.com/lyc8503/UptimeFlare) 是用来实时监控自己服务是否可运行的一个项目，其实我也不咋用得到，但是可以用来爽爽（x

![Panel](https://github.com/lyc8503/UptimeFlare/blob/main/docs/desktop.png?raw=true "Panel")

看着满屏的数据让人赏心悦目

~~（呕，Strong男~~

## Bark

这个 [cwxiaos/bark-worker](https://github.com/cwxiaos/bark-worker) 是我最近部署的一个项目，它作为 `iOS` 端的 Bark 应用实现自定义的推送通知，依赖 `APNs` 实现系统级的推送服务。

原本我的想法是想根据 `WebHook` 实现自建推送都集成在一起，不过目前还没实现，等实现了再补充吧

![App](https://camo.githubusercontent.com/c7b3d7ac64480a389fe9f0ccb5b89d938f8569d0b790aa51ab8bbb6c4734e9c7/68747470733a2f2f7778342e73696e61696d672e636e2f6d77323030302f30303372596671706c7931677264316d65717276636a3630626930387a74396930322e6a7067 "App")

## Bitwarden

实际上是非官方版的 [dani-garcia/vaultwarden](https://github.com/dani-garcia/vaultwarden) ，由 `Rust` 编写并且支持官方的所有接口，比起官方的服务器自建更好一些吧~~（并不~~

之前我的密码管理比较随意，各个设备浏览器之间也不同步，所以心血来潮做了统一规整，而且它对 `iOS` 支持也很好，可以直接代替系统自带的密码管理器，对 `2FA` 和 `PassKey` 的支持也非常到位，多端同步加上浏览器插件用起来非常得心应手。（再也不用一直掏手机扫码或者看验证码了hhhhhh

由于是密码相关所以安全性和备份异常重要，我是对整个数据采取**定期备份**，并且及时**更新**版本避免漏洞，另外注意不要过于依赖，自建由完全丢失的风险，非专业人士还是建议直接用官方服务器吧。
