#linux系统下的文件编程
##本内容是自己学习的笔记，主要内容参考与 朱文伟 李建英 著的书籍《linux C与C++ 一线开发实践》
***
<pre style="background:#222;color:#fff">
文件描述符
一个进程启动时，默认会打开三个文件 标准输入 标准输出 标准错误
对应的文件描述符为 0-(STDIN_FILENO) 1-(STDOUT_FILENO)  2-(STDERR_FILENO) 这些常量定义在unistd.h头文件中
而stdin stdout stderr 都是文件的指针  这两者之间可以相互转换
4.1
文件转换:
int fileno(FILE *stream);  //将文件指针转换为文件描述符
FILE *fdopen(int fd,const char *mode) //将文件描述符转换为文件指针
具体可以通过man文档

4.2打开和创建文件
#include<fcntl.h>
int open(const char *pathname, int flags);
int open(const char *pathname,int flags,mode_t mode);
flags:文件打开方式    mode只有创建文件才能使用这个 指定文件的访问权限
int creat(const char*pathname, mode_t mode) (如果文件之前存在 ，则会覆盖原来的文件）
int open(file,O_WRONLY|O_CREAT|O_TRUNC,mode)
4.3关闭文件
#include<unistd.h>
int close(int fd);执行成功返回文件描述符  失败返回-1

4.4:循环建立文件 最多可以键65534个文件描述符

4.5：读取文件数据
#include<unistd.h>
ssize_t read(int fd,void *buf,size_t count);
将fd所指的文件中传送count个字节到buf指针所指的内存出，返回的是实际读取到的字节数
如果返回0表示已经到达文件尾或没有可以读取的数据。文件的读写位置会随着读取的字节移动
如果读取成功返回的是实际读到的字节数  最好与count坐标叫，若返回的字节数比count小，则有可能
读到了文件尾或者read()被信号中断了读取动作。当有错误发生时，则返回-1 错误代码存入errno中
常见的错误代码 EINTR：调用被信号所中断 EAGAIN当使用不可阻挡I/O时若无数据可读，则返回此值
EBADF：参数fd为非有效的文件描述符，或当前文件已关闭

4.6:向文件中写入文件
#include<unistd.h>
ssize_t write(int fd,const void *buf,ssize_t count);
一般可以用count-->strlen(buf)得到buf长度
如果函数执行成功，则返回实际写入数据的字节数  当有错误发生返回-1
此时以用errno查看 常见
EINTR此调用被信号所中断
EADF参数fd时非有效的文件描述符 或该文件已经被关闭


4.7：设定文件偏移量
#include<unistd.h>
off_t lseek(int fd,off_t offset, int whence);
按照操作模式whence和偏移量的带下off_t重新设定文件偏移量 如果lseek()函数操作成功就返回新的文件的偏移的值，如果失败就返回-1 
有文件的偏移量可以为复制，因此lseek()是否操作成功，不要使用小于0的判断要使用== -1 来判断
whence:SEEK_SET offset 为相对文件开始处的位置
whence:SEEK_CUR offset为相对当前文件的位置
whence=SEEK_END时  offset为相对文件结尾的值

4.8获取文件状态
经常要用到文件的一些特征值，如文件的所有者  文件的修改时间  文件的大小
stat() fstat()  lstat()都可以获取文件的状态
int stat(const char *path,struct stat *buf);
int fstat(int filedes,struct stat *buf);
int lstat(const char* path,struct stat *buf);
path->文件的路径(包含文件名） filedes:fd  
函数执行成功时返回0 执行失败时返回-1
fstat() 与其他两个区别使用的时文件描述符，时需要我们先使用open系统调用后得到的
stat与lstat区别在于 当文件时一个符号链接时，lstat返回的时该符号链接本身的消息 ，stat返回的是该链接所指向的文件的信息
stat结构体就时文件类型的具体信息集合
struct stat
{
	mode_t st_mode; //文件对应的模式、文件、目录等
	ino_t st_ino;	//inode节点号
	dev_t st_dev;	//设备号码
	dev_t st_rdev;	//特殊设备号码
	nlink_t st_nlink;//文件的链接数
	uid_t st_uid;	//文件的所有者
	gid_t st_gid;	//文件的所有者对应的所有组	
	off_t st_size;	//普通文件对应的文件字节数
	time_t st_atime;	//文件最后被访问的时间
	time_t st_mtime;	//文件内容最后被修改的时间
	time_t st_ctime;	//文件状态改变的时间
	blksize_t st_blksize;	//文件内容对应的块大小
	blkcnt_t st_blocks;		//文件内容对应的块数量
}


4.9文件锁定
当多个用户共同使用时，操作一个文件Linux采用的是给文件上锁，来避免共享资源产生竞争
文件锁分为建议性锁和强制性锁
建议性锁就是给文件上锁后，只在文件上设定锁的标识，其他进程对这个文件进程操作时，可以检测锁的存在，但是这个锁不能够阻止其他进程操作这个文件，不一定能拦住。
强制性锁就是给文件上锁后，当其他进程要对这个文件进行不兼容的操作（比如上了读锁，零一个进程要写）系统内核就会阻塞后来的进程，知道第一个进程将锁解开。一般情况，内核和系统都要使用强制性锁，这样可以防止一些破坏性的操作。例如执行open read write内核会检测该文件是否被加了强制性锁，如果加了强制性锁，这些操作就会失败。
同时：对文件加锁时原子性的。另外有fork产生的子进程不继承父进程锁设置的锁，这就意味着如一个进程得到一把锁，然后fork那么对与父进程获得的锁而言，子进程被视为另一个进程。对于从父进程处继承过来的任一描述符，子进程都需要调用fcntl才能获得他自己的锁 
fcntl不仅可以对整个文件上锁，而且可以对文件某一记录上锁，成为记录锁
fcntl
#include<unistd.h>
#include<fcntl.h>
int fcntl(int fd,int cmd, struct flock *lock)

</pre>
