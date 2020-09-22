# 1. 什么是CRLF和LF

由于历史的原因，各种不同的操作系统在处理行尾结束符采取了不同的处理方法。

CRLF 是carriagereturnline feed的缩写。中文意思是回车换行。LF是line feed的缩写，中文意思是换行。

CRLF->Windows-style

LF->Unix Style

CR->Mac Style

CRLF表示句尾使用回车换行两个字符(即我们常在Windows编程时使用"\r\n"换行)

LF表示表示句尾，只使用换行.

CR表示只使用回车.



# 2. Git 对CRLF和LF换行符的处理

- 首先说一下Git为什么会对换行符做处理

在Git的帮助页面给出了很好的解释。

Reference From:https://help.github.com/articles/dealing-with-line-endings

If you're using Git to collaborate with others on GitHub, ensure that Git isproperly configured to handle line endings.

Every time you press return on your keyboard you're actuallyinserting an invisible character called aline ending. Historically, differentoperating systems have handled line endings differently.

When you view changes in a file, Git handles line endings in its own way.Since you're collaborating on projects with Git and GitHub, Git mightproduce unexpected results if, for example, you're working on a Windows machine,and your collaborator has made a change in OS X.

大概意思就是不同的系统会产品不同的换行符，而不同的人可能用不同的系统；为了更高效的与他人合作，Git会对换行符做统一的处理；

在学习git软件，安装git到configuring the lien ending conversion时，有三个选项。

a. Checkout Windows-style,commit Unix-style line endings.

b.Checkout as-is,commit Unix-style line endings.

c.Checkout as-is,commit as-is line endings.

这里面讲到了做两个操作(Checkout,Commit)的三种处理line endings的操作(Windows-style,Unix-style,As-is)。

# 3. Git autocrlf 的使用

在Git通过下面的命令配置

$git config --global core.autocrlf true
\# Configure Git on Windows to properly handle line endings

\# Configure Git on Windows to properly handle line endings
解释：core.autocrlf是git中负责处理line endings的变量，可以设置三个值--true,inout,false.

设置成三个值会有什么效果呢？

If core.autocrlf is set to true, that means that any time you add a file to the git repo that git thinks is a text file, it will turn all CRLF line endings to just LF before it stores it in the commit.。

设置为true，添加文件到git仓库时，git将其视为文本文件。他将把crlf变成lf。

If core.autocrlf is set to false, no line-ending conversion is ever performed, so text files are checked in as-is. This usually works ok。

设置为false时，line-endings将不做转换操作。文本文件保持原来的样子。

设置为input时，添加文件git仓库时，git把crlf编程lf。当有人Check代码时还是lf方式。因此在window操作系统下，不要使用这个设置。

转换如表格：

| core.autocrlf | file to commit | repository | checked out file |
| ------------- | -------------- | ---------- | ---------------- |
| true          | X              | LF         | CRLF             |
| input         | X              | LF         | LF               |
| false         | X              | X          | X                |

X 为LF或者CRLF