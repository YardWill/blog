```
$ vimtutor //进入vim的教程模式
```

### One Chapter
1. 使用 h、j、k、l来进行光标的移动
2. 使用vim filename 编辑文本
3. 使用 q! 强制退出 使用 wq 保存并退出
4. 删除光标所在字母 x
5. i 进入编辑模式，esc进入正常模式

### Two Chapter
1. dw delete word 删除单个单词
2. d$ 删除从光标到句末到所有单词
3. de 删除光标到单词末
4. 2w 2e 跳过n个单词
5. d2w 删除两个n个单词
6. dd 删除整行 2dd 删除n个整行
7. u 撤销修改 

### Three Chapter
1. p 粘贴
2. r x replace 单词替换
3. ce 清除光标到单词末尾的内容
4. c 3w  c$  删除几个单词 删除到行末

### Four Chapter
1. gg 开头 G 结尾
2. / 搜索 使用n查找下一个
3. % to find a matching ),], or }
4. :s/old/new/g 替换

### Five Chapter
1. :! 命令行模式
2. :w filename 保存文件
3. v 选择
4. :r filename 插入文件内的内容

### Fix Chapter
1. o 光标至下一行
2. a 插入 和e配合使用
3. R 替换
4. v选择 y复制 p粘贴