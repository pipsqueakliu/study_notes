#linux系统下的文件编程
***
<pre style="background:#222;color:#fff">
文件描述符
一个进程启动时，默认会打开三个文件 标准输入 标准输出 标准错误
对应的文件描述符为 0-(STDIN_FILENO) 1-(STDOUT_FILENO)  2-(STDERR_FILENO) 这些常量定义在unistd.h头文件中
而stdin stdout stderr 都是文件的指针  这两者之间可以相互转换
1:
int fileno(FILE *stream);  //将文件指针转换为文件描述符
FILE *fdopen(int fd,const char *mode) //将文件描述符转换为文件指针

</pre>
