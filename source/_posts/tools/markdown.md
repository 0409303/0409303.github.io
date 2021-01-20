---
    title: markdown语法
    categories: tools
---

## 参考资料
全部数学符号 https://katex.org/docs/supported.html

https://www.cnblogs.com/caomingpei/p/9760652.html

## 数学公式
inline math：`$x^{y^z}=(1+e^x)^{-2xy^w}$`

block math:
```math
\frac{7x+5}{1+y^2}
```

$$
\mathbf{V}_1 \times \mathbf{V}_2 =  \begin{vmatrix} 
\mathbf{i} & \mathbf{j} & \mathbf{k} \\
\frac{\partial X}{\partial u} &  \frac{\partial Y}{\partial u} & 0 \\
\frac{\partial X}{\partial v} &  \frac{\partial Y}{\partial v} & 0 \\
\end{vmatrix}
${$tep1}{\style{visibility:hidden}{(x+1)(x+1)}}
$$

## 文本
#### 上下标
1. 上标：n^2^
2. 下标：n~2~

#### 颜色
Markdown文字添加颜色
https://www.jianshu.com/p/aefd0e50b802

## 流程图
```
graph TD
    a-->b
    a-->c
    d-->e
```
```flow
st=>start: Start|past:>http://www.google.com[blank]
e=>end: End:>http://www.google.com
op1=>operation: My Operation|past
op2=>operation: Stuff|current
sub1=>subroutine: My Subroutine|invalid
cond=>condition: Yes
or No?|approved:>http://www.google.com
c2=>condition: Good idea|rejected
io=>inputoutput: catch something...|request

st->op1(right)->cond
cond(yes, right)->c2
cond(no)->sub1(left)->op1
c2(yes)->io->e
c2(no)->op2->e
```


# 下面是通过html语法完成的，非markdown自带功能
## 改变文字样式
<p style="text-align:center;color:#1e819e;font-size:1.3em;font-weight:bold;">
改变文字的大小、颜色和对齐方式
</p>

## 自定义表格样式
<table>
    <tr>
        <td rowspan="7"> 文件状态：
            <br/> [√] 草稿
            <br/> [√] 正在修改
            <br/> [√] 正式发布 </td>
        <td>文件名称：</td>
        <td>LSGO股票交易策略分析软件 Use Case Model</td>
    </tr>
    <tr>
        <td>当前版本：</td>
        <td>1.0.0</td>
    </tr>
    <tr>
        <td>作 者：</td>
        <td>马燕鹏</td>
    </tr>
    <tr>
        <td>创建日期：</td>
        <td>2018-07-15</td>
    </tr>
    <tr>
        <td>最后更新：</td>
        <td></td>
    </tr>
    <tr>
        <td>密 级：</td>
        <td>开源系统</td>
    </tr>
    <tr>
        <td>版权说明：</td>
        <td>遵守 GPL V3协议</td>
    </tr>
</table>

## 折叠内容
<details>
<summary>explode 与 lateral view 对比</summary>
<pre><code>
select user_coupon_id, explode(split('0,1', ',')) as tag
from mart_waimai.aggr_act_ord_use_coupon_dd
where dt='20200920'
limit 10
</code></pre>
</details>

## 支持html标签


# 参考资料
1. [完整整理版](https://blog.csdn.net/witnessai1/article/details/52551362)
2. [流程图详细教程](https://mermaidjs.github.io/)
3. [编辑数学公式](http://blog.csdn.net/lk7688535/article/details/52528307)
4. [Markdown 高级技巧](https://www.runoob.com/markdown/md-advance.html)
5. [Hexo下的Markdown语法](https://www.dazhuanlan.com/2019/10/16/5da6b7885b94f/)
6. [【hexo博客进阶】1.Markdown语法](https://www.cnblogs.com/huanhao/p/markdown.html)
7. [markdown语法](https://www.appinn.com/markdown/)
8. [如何让hexo使用流程图](https://segmentfault.com/q/1010000009073806?utm_source=sf-similar-question)
9. [Hexo优化 --- 利用 Markdown 语法画流程图](http://www.zhaojun.im/hexo-flowchart/)

# markdown工具
1. [Typora](https://www.typora.io/)