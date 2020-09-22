1. python代码在代码上面加上#%% ，用vscode 打开，即可作为code cell 运行

   ![1573031499218](C:\Users\wpzhou\AppData\Roaming\Typora\typora-user-images\1573031499218.png)

2. 也可以直接编辑ipynb 的文件，插入代码块或者markdown 块， 用来写博客很好用

   ![1573031581899](C:\Users\wpzhou\AppData\Roaming\Typora\typora-user-images\1573031581899.png)

3. 使用ipynb文件分析完成之后，可以直接转换成[1] 中的python脚本

![1573031695180](C:\Users\wpzhou\AppData\Roaming\Typora\typora-user-images\1573031695180.png)

4. 将ipynb 其他格式（asciidoc, custom, html, latex, markdown, notebook, pdf, python, rst, script, slides）的文档

   ```shell
   jupyter nbconvert --to markdown demo.ipynb
   ```

   

   