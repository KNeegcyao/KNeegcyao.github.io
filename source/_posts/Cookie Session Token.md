---
title: '搞懂Cookie,Session,Token三种认证授权方式'
tags:
  - 基础知识
  - java
categories:
  - Web开发
top_img: >-
  'https://haowallpaper.com/link/common/file/previewFileImg/15918089447001472'
cover: https://cdn.jsdelivr.net/gh/KNeegcyao/picdemo/img/image-20241216163229547.png
abbrlink: 4ed1237b
date: 2024-12-16 16:55:59
---
# Cookie，Seesion和Token区别及用途

![image-20241216163229547](https://cdn.jsdelivr.net/gh/KNeegcyao/picdemo/img/image-20241216163229547.png)

## 简介

Cookie、Session、Token 和 JWT（JSON Web Token）都是用于在网络应用中进行身份验证和状态管理的机制。虽然它们有一些相似之处，但在实际应用中有着不同的作用和特点。

## Cookie

**定义：**

- HTTP 是无状态的协议（对于事务处理没有记忆能力，每次客户端和服务端会话完成时，服务端不会保存任何会话信息）：每个请求都是完全独立的，服务端无法确认当前访问者的身份信息，无法分辨上一次的请求发送者和这一次的发送者是不是同一个人。所以服务器与浏览器为了进行会话跟踪（知道是谁在访问我），就必须主动的去维护一个状态，这个状态用于告知服务端前后两个请求是否来自同一浏览器。而这个状态需要通过 cookie 或者 session 去实现。
- cookie 存储在客户端：cookie 是服务器发送到用户浏览器并保存在本地的一小块数据，它会在浏览器下次向同一服务器再发起请求时被携带并发送到服务器上。
- cookie 是不可跨域的：每个 cookie 都会绑定单一的域名，无法在别的域名下获取使用，一级域名和二级域名之间是允许共享使用的（靠的是 domain）。

**使用过程：**

![image-20241216163732161](https://cdn.jsdelivr.net/gh/KNeegcyao/picdemo/img/image-20241216163732161.png)

![image-20241216164101074](https://cdn.jsdelivr.net/gh/KNeegcyao/picdemo/img/image-20241216164101074.png)

## Seesion

**定义：**

- session 是另一种记录服务器和客户端会话状态的机制
- session 是基于 cookie 实现的，session 存储在服务器端，sessionId 会被存储到客户端的cookie 中

**使用过程：**

![image-20241216164427290](https://cdn.jsdelivr.net/gh/KNeegcyao/picdemo/img/image-20241216164427290.png)

**session 认证流程：**

- 用户第一次请求服务器的时候，服务器根据用户提交的相关信息，创建对应的 Session
- 请求返回时将此 Session 的唯一标识信息 SessionID 返回给浏览器
- 浏览器接收到服务器返回的 SessionID 信息后，会将此信息存入到 Cookie 中，同时 Cookie 记录此 SessionID 属于哪个域名
- 当用户第二次访问服务器的时候，请求会自动判断此域名下是否存在 Cookie 信息，如果存在自动将 Cookie 信息也发送给服务端，服务端会从 Cookie 中获取 SessionID，再根据 SessionID 查找对应的 Session 信息，如果没有找到说明用户没有登录或者登录失效，如果找到 Session 证明用户已经登录可执行后面操作。

![image-20241216164852355](https://cdn.jsdelivr.net/gh/KNeegcyao/picdemo/img/image-20241216164852355.png)

## Token

**使用流程（JWT）：**

![image-20241216165331779](https://cdn.jsdelivr.net/gh/KNeegcyao/picdemo/img/image-20241216165331779.png)

**JWT 认证流程：**

- 前端发送登录请求，后端进行认证，认证成功后，会创建一个JWT的字符串，包含三段信息。
- 然后把创建好的JWT返回到前端，前端拿到JWT的token，对token分割拿到第二段，第二段是数据的载体，通过base64的方式解码即可。
- 登录成功后，后续的请求，只需要在请求头中设置一个跟后端约定的一个属性```Authorization：token``` ,然后再把响应的JWT Token传入到后端，后端拿到Token进行解密，验证signature,检验成功信息没有被篡改，即可放行，正常响应。

JWT不需要前后端的存储，只是一个加密的字符串，对于集群和前后端分离的架构来说是非常方便的。