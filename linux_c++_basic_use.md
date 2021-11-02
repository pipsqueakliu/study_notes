#linux_C++_basic_use
<pre>
针对c语言：
预处理:主要针对#开头的的预编译将其替换
gcc -E test.c -o test.i 

编译：一系列的词法分析 语法分析 语义分析 优化后生成汇编代码文件
gcc -S test.i -o test.s

汇编:将汇编代码变为机器剋执行的二进制代码
gcc -c test.s -o test.o

链接：链接其他的文件生成可执行文件
gcc test.o -o test

关于gcc的用法
gcc [选项] 准备编译的文件 [选项] [目标文件]
gcc --help查看具体指令如何使用

关于gcc -I dirPath （头文件的搜索顺序）
如果源代码中用尖括号包含头文件
step1:先寻找-I 指定的路径中搜索所需的头文件，若没找到进入step2
step2：寻找标准默认路径/usr/local/include中搜索，若没找到进入step3
step3:寻找/usr/include 若没有知道则报错
注：尖括号包含的文件不会在当前工作目录下搜索，即使当前工作目录由所需要的头文件

如果源代码中用双引号包含头文件
step1:gcc先在当前工作目录（和源文件同一目录）进行查找 如果没有找到进入step2
step2:gcc在-I指定的路径中查找 如果没有找到进入step3
step3:gcc在默认标准默认路径/usr/local/include进行查找 如果没有找到进入step4
step4:gcc在标准默认路径/usr/include中进行查找 如果没有找到就会报错

也可以使用简单的方法
gcc [srcfile] -include [headfile] ---headfile（直接头文件可以用绝对路径 也可以用相对路径）

关于-l 用来链接共享库（动态链接库）
gcc test.cpp -lstdc++ -o test 链接C++标准库
</pre>
<pre>
gdb的调试使用
gdb时加载一个可执行文件  该文件必须由gcc  -g  产生
gcc -g test.cpp -o test
gdb  启动gdb
file test  加载可执行文件
进行调试

调试方法：
1： list 不带参数显示源代码10行内容
输入list或者l
 2：list 带参数 n 就是显示n前五行与后四行的内容
3：list 带两个参数n1 n2 就是显示从n1行到n2行
4：list + funcname 就是显示该函数附近的源代码内容


run命令
run boy girl 就是将boy 和girl作为参数传输
如果直接用run不带参数就是使用上一次使用的参数
如果需要重新设计参数的话需要使用set args dad mum 之后run

break设置断点
b(break) linenumber 在源代码的某处设置断点
b(break) myfunc   在源代码的某函数出设置断点
</pre>