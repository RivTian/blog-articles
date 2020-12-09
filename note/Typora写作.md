---
title: "Typora写作"
date: 2020-07-30T16:04:42+08:00
draft: false
tags:
  - Typora
topics:
  - 
---

> 参考：
>
> * [如何高效地编写同步博客](https://www.cnblogs.com/stulzq/p/9043632.html)
> * [官方文档](https://support.typora.io/Upload-Image/#upload-automatically-when-insert-images)

## 图片插入

我们在网页复制图片，或者插入本地图片，亦或者使用QQ截图，插入到我们的博客中时，可以通过下面的设置，将目标图片复制到与我们博客同级的`assets`目录中

![](https://cdn.jsdelivr.net/gh/Kanna-jiahe/blogimage/img/20200801160642.png)



但是利用同级目录比较麻烦，还是使用PicGo图床搭配Typora舒适感更好（粘贴图片直接上传）

[手把手教你用Typora自动上传到picgo图床【教程与排坑】](https://zhuanlan.zhihu.com/p/114175770)

图床使用由Gitee 更新为 Github：[Click Here](https://www.cnblogs.com/RioTian/p/13404542.html)

![测试](https://cdn.jsdelivr.net/gh/Kanna-jiahe/blogimage/img/20200801160649.jpg)



### 图片居中

样例示意

<div align=center><img src="https://gitee.com//riotian/blogimage/raw/master/img/20201123190654.gif"/></div>

```html
<div align=center><img src="https://gitee.com//riotian/blogimage/raw/master/img/20201123190654.gif"/></div>
```



## 同步到GitHub

我们使用**Typora**编辑器编写的博客可以非常轻松的同步到`Github`，可以直接使用`Git`等工具上传到我们的`Github`仓库 。



# Markdown常用数学符号&公式



| 符号                                           | 代码                                                         | 描述       |
| :--------------------------------------------- | :----------------------------------------------------------- | ---------- |
| $\frac{1}{3} 与 \cfrac{1}{3}$                  | 分数使用`\frac{分母}{分子}`这样的语法，不过推荐使用`\cfrac`来代替`\frac`，显示公式不会太挤。 | 分数       |
| $\sim$                                         | `$\sim$`                                                     | 波浪号     |
| $\sum$                                         | `$\sum$`                                                     | 求和公式   |
| $\sum_{i=0}^n$                                 | `$\sum_{i=0}^n$`                                             | 求和上下标 |
| $\times$                                       | `$\times$`                                                   | 乘号       |
| $\pm$                                          | `$\pm$`                                                      | 正负号     |
| $\div$                                         | `$\div$`                                                     | 除号       |
| $ \mid$                                        | `$\mid$`                                                     | 竖线       |
| $\cdot$                                        | `$\cdot$`                                                    | 点         |
| $\circ$                                        | `$\circ$`                                                    | 圈         |
| $\ast $                                        | `$\ast $`                                                    | 星号       |
| $ \bigotimes$                                  | `$\bigotimes$`                                               | 克罗内克积 |
| $\bigoplus$                                    | `$\bigoplus$`                                                | 异或       |
| $\leq$                                         | `$\leq$`                                                     | 小于等于   |
| $\geq$                                         | `$\geq$`                                                     | 大于等于   |
| $ \neq$                                        | `$\neq$`                                                     | 不等于     |
| $\approx$                                      | `$\approx$`                                                  | 约等于     |
| $ \prod$                                       | `$\prod$`                                                    | N元乘积    |
| $ \coprod$                                     | `$\coprod$`                                                  | N元余积    |
| $ \cdots$                                      | `$\cdots$`                                                   | 省略号     |
| $\int$                                         | `$\int$`                                                     | 积分       |
| $\iint$                                        | `$\iint$`                                                    | 双重积分   |
| $\oint$                                        | `$\oint$`                                                    | 曲线积分   |
| $ \infty$                                      | `$\infty$`                                                   | 无穷       |
| $\nabla$                                       | `$\nabla$`                                                   | 梯度       |
| $\because$                                     | `$\because$`                                                 | 因为       |
| $\therefore$                                   | `$\therefore$`                                               | 所以       |
| $\forall$                                      | `$\forall$`                                                  | 任意       |
| $\exists$                                      | `$\exists$`                                                  | 存在       |
| $\not=$                                        | `$\not=$`                                                    | 不等于     |
| $ \not>$                                       | `$\not>$`                                                    | 不大于     |
| $ \leq$                                        | `$\leq$`                                                     | 小于等于   |
| $\geq$                                         | `$\geq$`                                                     | 大于等于   |
| $ \not\subset$                                 | `$\not\subset$`                                              | 不属于     |
| $\emptyset$                                    | `$\emptyset$`                                                | 空集       |
| $\in$                                          | `$\in$`                                                      | 属于       |
| $\notin$                                       | `$\notin$`                                                   | 不属于     |
| $\subset$                                      | `$\subset$`                                                  | 子集       |
| $\subseteq$                                    | `$\subseteq$`                                                | 真子集     |
| $\bigcup$                                      | `$\bigcup$`                                                  | 并集       |
| $\bigcap$                                      | `$\bigcap$`                                                  | 交集       |
| $\bigvee$                                      | `$\bigvee$`                                                  | 逻辑或     |
| $\bigwedge$                                    | `$\bigwedge$`                                                | 逻辑与     |
| $\biguplus$                                    | `$\biguplus$`                                                | 多重集     |
| $\bigsqcup$                                    | `$\bigsqcup$`                                                |            |
| $\hat{y}$                                      | `$\hat{y}$`                                                  | 期望值     |
| $\check{y}$                                    | `$\check{y}$`                                                |            |
| $\breve{y}$                                    | `$\breve{y}$`                                                |            |
| $\overline{a+b+c+d}$                           | `$\overline{a+b+c+d}$`                                       | 平均值     |
| $\underline{a+b+c+d}$                          | `$\underline{a+b+c+d}$`                                      |            |
| $\overbrace{a+\underbrace{b+c}_{1.0}+d}^{2.0}$ | `$\overbrace{a+\underbrace{b+c}_{1.0}+d}^{2.0}$`             |            |
| $\uparrow$                                     | `$\uparrow$`                                                 | 向上       |
| $\downarrow$                                   | `$\downarrow$`                                               | 向下       |
| $\Uparrow$                                     | `$\Uparrow$`                                                 |            |
| $\Downarrow$                                   | `$\Downarrow$`                                               |            |
| $\rightarrow$                                  | `$\rightarrow$`                                              | 向右       |
| $\leftarrow$                                   | `$\leftarrow$`                                               | 向左       |
| $\Rightarrow$                                  | `$\Rightarrow$`                                              | 向右箭头   |
| $\Longleftarrow$                               | `$\Longleftarrow$`                                           | 向左长箭头 |
| $\longleftarrow$                               | `$\longleftarrow$`                                           | 向左单箭头 |
| $\longrightarrow$                              | `$\longrightarrow$`                                          | 向右长箭头 |
| $\Longrightarrow$                              | `$\Longrightarrow$`                                          | 向右箭头   |
| $\alpha$                                       | `$\alpha$`                                                   |            |
| $\beta$                                        | `$\beta$`                                                    |            |
| $\gamma$                                       | `$\gamma$`                                                   |            |
| $\Gamma$                                       | `$\Gamma$`                                                   |            |
| $\delta$                                       | `$\delta$`                                                   |            |
| $\Delta$                                       | `$\Delta$`                                                   |            |
| $\epsilon$                                     | `$\epsilon$`                                                 |            |
| $\varepsilon$                                  | `$\varepsilon$`                                              |            |
| $\zeta$                                        | `$\zeta$`                                                    |            |
| $\eta$                                         | `$\eta$`                                                     |            |
| $\theta$                                       | `$\theta$`                                                   |            |
| $\Theta$                                       | `$\Theta$`                                                   |            |
| $\vartheta$                                    | `$\vartheta$`                                                |            |
| $\iota$                                        | `$\iota$`                                                    |            |
| $\pi$                                          | `$\pi$`                                                      |            |
| $\phi$                                         | `$\phi$`                                                     |            |
| $\Phi$                                         | `$\Phi$`                                                     |            |
| $\psi$                                         | `$\psi$`                                                     |            |
| $\Psi$                                         | `$\Psi$`                                                     |            |
| $\omega$                                       | `$\omega$`                                                   |            |
| $\Omega$                                       | `$\Omega$`                                                   |            |
| $\chi$                                         | `$\chi$`                                                     |            |
| $\rho$                                         | `$\rho$`                                                     |            |
| $\omicron$                                     | `$\omicron$`                                                 |            |
| $\sigma$                                       | `$\sigma$`                                                   |            |
| $\Sigma$                                       | `$\Sigma$`                                                   |            |
| $\nu$                                          | `$\nu$`                                                      |            |
| $\xi$                                          | `$\xi$`                                                      |            |
| $\tau$                                         | `$\tau$`                                                     |            |
| $\lambda$                                      | `$\lambda$`                                                  |            |
| $\Lambda$                                      | `$\Lambda$`                                                  |            |
| $\mu$                                          | `$\mu$`                                                      |            |
| $\partial$                                     | `$\partial$`                                                 |            |
| $\lbrace \rbrace$                              | `$\lbrace \rbrace$`                                          |            |
| $\overline{a}$                                 | `$\overline{a}$`                                             |            |