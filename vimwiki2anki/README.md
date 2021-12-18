## 使用方法

+ 把vimwiki2anki目录加入`$PATH`环境变量(向bashrc中追加一行)
  ```
  #注意把 $HOME/vimwiki/my-shell-script 改为自己的文件夹路径
  #把存储自己所有脚本的父文件夹my-shell-script及其子文件(对应不同项目)夹加入$PATH
  export PATH=$PATH:$(find $HOME/vimwiki/my-shell-script -maxdepth 1 -type d | paste -sd ":" -)
  ```
+ 直接使用

    ```bash
    #脚本接受一个文件名作为输入参数
    vimwiki2anki demo.md
    #转换后的文件保存在原文件的同一目录下，名为demo.md.manki
    ```

+ 在vim中使用
  ```
  # 在vimrc中加入快捷指令，normal模式下输入m0会把当前m1格式的vimwiki文件demo.md复制到demo.md.manki
  # 并转为可导入anki的笔记
  nmap m0 :!vimwiki2anki %:t<CR><CR>
  ```
+ 卡片导入anki：使用自带的填空题模板，导入卡片时以`！(中文感叹号)为分隔符号`，导入时第一字段对应到`文字`，第二字段对应到`标签`。之后在anki中使用[hierachical tag2](https://ankiweb.net/shared/info/594329229)插件，就能按标签查看对应章节的卡片。

## book-->vimwiki-->anki
+ 书籍通过目录将小块内容联系起来，层级结构一般来说有章-节-小节。通过vimwiki记笔记，可以使用相似的层级结构，但一本书的笔记文件不宜太多，单个文件既包含具体的内容又有一定的结构，可以将教材中一节的笔记放在一个文件中。
+ 使用思维导图来记录书籍的结构信息，连接不同的层级(章/节)，可以通过根(书籍名称)向下遍历/查看所有笔记内容。在笔记中绘制思维导图时，至多有三层结构。
+ 可使用anki进行复习的vimwiki笔记结构。
    + index.md
        + 整本书的思维导图(本书-章-节)
        + 指向不同章目录(ch#_index)的链接
    + ch0_index.md
        + 本章的思维导图(本章-节-小节(可选))
        + 指向不同节文件(ch#_se#)的链接
    + ch0_se0.md
        + 一级标题例子(标题必须是这个格式)：`# /ch1(第一章)/se1(第一节)`
        + 本节的思维导图(节-小节/block)
        + 一个或多个block
            + block标题例子(标题必须是这个格式)：`## /bl1(第一块)`
            + block就是一个无序列表，列表中每一项都是一条anki-note。
                + anki-note通过挖空的方式对知识进行再加工，通过给不同部分加上不同的挖空标签(<c1></c>/<c2></c>)，每次隐去一个标签对应的内容，一条anki-note可以产生多个card。anki的最小记忆单元是card，这样一对多的方式,让相同的信息在多个卡片中重复出现，也有利于之后的学习和测试。
                + anki-note标签类别：
                    + m1(代码易读,md输出没有颜色):`其他内容，<c1>挖空内容</c>，其他内容。`
                    + m2(代码不易读,md不同的空有不同的颜色):`其他内容，<font color=#FF6666>挖空内容</font>，其他内容。`
## vimwiki2anki-shell-script
+ 目标文件格式(导入anki批量制作卡片,具体请看[anki](https://docs.ankiweb.net/))：分为两部分，第一部分是主要内容(挖过空的)，第二部分是内容在书籍中的位置信息(章节序号和名称)，两部分间使用中文感叹号！分隔。
    + 第一部分：`{{c1::card1上的空1}}`测试测试`{{c1::card1上的空2}}`，`{{c2::card2上的空1}}`。
    + 第二部分：ch#(章名)/se#(节名)/bl#(block名)
+ 脚本思路(仅是大概思路，具体实现请看代码)
    1. 先复制m1类别的vimwiki文件，并命名为.anki。
    ```bash
    newfilename=${filename}.manki$
    cp $filename $newfilename
    ```
    2. 删除文件中的所有代码块。
    ```bash
    str=$(grep -n "\`\`\`" 文件名称 | cut -d: -f1)#查找文件中代码块标志所在的行数
    arr=(${str//,/})#把字符串截取为数组arr
    #使用delete_lines从后往前，删除代码块所在的行
    bash delete_lines $newfilename $begin $end
    ```
    3. 查找本节(文件)内章节ch#_se#信息和block信息(标题和位置)。
    ```bash
    #查找章节
    headstr=$(grep -n "# /ch" $newfilename | cut -d/ -f2)$(grep -n "# /ch" $newfilename | cut -d/ -f3)
    #查找block
    blockstr=$(grep -n "## /bl" $newfilename | cut -d/ -f2)$
    blocklinestr=$(grep -n "## /bl" $newfilename | cut -d: -f1)
    ```
    4. 把anki-note笔记所属章节信息加入行尾。
    ```bash
    #使用add_info脚本把章节信息加入对应笔记的行尾
    info=${headstr}${blockheadarr[$j]}
    bash add_info $newfilename ${blocklinearr[$j]} ${blocklinearr[$j+1]} $info
    ```
    5. 删除非anki-note笔记行，把标签改为anki内使用的格式
    ```bash
    #在bash脚本中使用vim命令
    #删除非anki-note行
    g!/+\s/d
    #删除anki-note行首的`+ `
    g/+\s/normal 0d2l
    #标签替换
    %s/<c1>/{{c1::/g
    ```
## 参考

+ David Chen([B站](http,,dfis://space.bilibili.com/13081489),[github](https://github.com/theniceboy))录制的[vim教程](https://www.bilibili.com/video/BV164411P7tw)
