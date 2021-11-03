#linux系统下的文件编程
##本内容是自己学习的笔记，主要内容参考与 朱文伟 李建英 著的书籍《linux C与C++ 一线开发实践》
***
##C语言下的文件编程
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
cmd可以是
F_GETLK:根据lock描述，决定是否上文件锁（或者记录所）
F_SETLK:设置lock描述的文件锁（或记录锁）
fcntl默认的是建议锁  如果想在linux中使用强制锁需要在root权限下
通过mount命令-o mand选项打开该机制
struct flock
{
	short int l_type;//锁定的状态
	short int l_whence;//决定l_start的位置
	off_t l_start;//锁定区域的开头位置
	off_t l_len;//锁定区域的大小
	off_t l_pid;//锁定动作的进程
};
l_type:F_RDLCK共享锁（读写锁）只读用  多个进程可以同时建立读取锁
F_WRLCK:独占锁（写入锁） 在任何时刻只能有一个进程建立写入锁
F_UNLCK：解除锁定
l_whence:
SEEK_SET:文件开始位置
SEEK_CUR:文件当前位置
SEEK_END;文件末尾位置
l_start:
l_len:加锁的长度 0为到文件末尾
l_pid:当前操作文件的进程ID号

加强制锁
df -hT 查看硬盘信息
-T选项表示查看文件系统类型 

##尚待跟进

4.11建立文件和内存映射
文件和内存映射--将普通文件映射到内存中，普通文件被映射到进程地址空间后，进程可以像访问普通内存一样对文件进行访问，不必在调用read或write等操作
void *mmap(void *start,size_t length,int prot,int flags,int fd,off_t offset);
参数start为映射区的起始位置通常为NULL（0）表示有系统自己决定映射到什么地址；length表示映射数据的长度
即文件需要映射到内存中的数据的大小 prot表示映射区保护方式，取以下某个值或者他们的组合
PROT_EXEC:映射区可被执行
PROT_READ:映射区可读取
PROT_WRITE:映射区可写入
PROT_NONO:映射区不可访问
具体参数可以通过man文档进行查看 man mmap
mmap()映射后，让用户程序直接访问设备内存  相比较在用户空间和内核空间相互复制数据，效率更高，在要求高性能的应用中比较常用 mmap映射内存必须是页面大小的整数倍，面向流的设备不能进行mmap  mmap的实现和硬件有关
</pre>
***
##C++方式下的文件IO编程
<pre style="background:#222;color:#fff">
1：打开文件
#include<fstream> 可读可写
#include<ifstream>从文件读取信息
#include<ofstream>向文件写入信息
void open(const char* filename,IOS::openmode mode);
mode:
ios::app 追加模式  所有写入都追加到文件末尾
ios:ate  文件打开后定位到文件末尾
ios:in   打开文件用于读取
ios:out 打开文件用于写入
ios::trunc  如果该文件已经存在，其内容将在打开文件之前被截断 即把文本长度设为0
等
可以几个组合使用
ofstream --> ios::out | ios::trunc
ifstream --> ios:in
fstream --> ios::in | ios::out

bool is_open() 该函数返回一个bool值 true表示文件已经打开  false表示失败

2：关闭文件
void close(); 是fstream  istream ostream 的成员函数

3：写入文件
使用ofstream <<      fstream<<
为了防止文件被覆盖需要先判断文件是否存在

#include <sys/stat.h>
#include <unistd.h>
#include <string>
#include <fstream>

inline bool exists_test0 (const std::string& name) {
    ifstream f(name.c_str());
    return f.good();
}

inline bool exists_test1 (const std::string& name) {
    if (FILE *file = fopen(name.c_str(), "r")) {
        fclose(file);
        return true;
    } else {
        return false;
    }   
}

inline bool exists_test2 (const std::string& name) {
    return ( access( name.c_str(), F_OK ) != -1 );
}

inline bool exists_test3 (const std::string& name) {
  struct stat buffer;   
  return (stat (name.c_str(), &buffer) == 0); 
}

# Results for total time to run the 100,000 calls averaged over 5 runs,

Method exists_test0 (ifstream): **0.485s**
Method exists_test1 (FILE fopen): **0.302s**
Method exists_test2 (posix access()): **0.202s**
Method exists_test3 (posix stat()): **0.134s**
摘自--CSDN--guotianqing
链接：https://blog.csdn.net/guotianqing/article/details/100766120

4:读取文件
istream >>   fstream >>

5:文件偏移位置
ostream& seekp(streampos pos);
ostream& seekp(streamoff off,ios::seek_dir dir);
istream& seekg(streampos pos);
istream& seekg(streamoff off,ios::seek_dir dir);
pos表示新的文件流指针位置值  off表示需要偏移的值 dir表示搜索的起始位置
dir:ios::beg  开始   ios::cur当前   ios::end 末尾

文件指针是一个整数值 指定了从文件的起始位置到指针所在位置的字节数 
fileObject.seekg(n) 定位到fileObject的第n个字节（ios::beg)
fileObject.seekg(n,ios::cur);将读指针从当前位置向后移动了n个字节
fileObject.seekg(n,ios::end) 将文件读指针从fileObject的末尾往回移动n个字节
fileObject.seekg(0,ios::end)定位到fileObject的末尾

5:状态标志符的验证
都是成员函数
bool bad()
如果读写过程中有错，就返回true
bool fail() :格式错误时也会返回true
bool eof(); 文件到达文件末尾时 返回true
bool good () 如果调用以上任何一个返回true此函数返回false
clear() 重置所有成员函数所检查的状态标志

6：读写文件数据块
ostream& write(char*buffer,streamsize size)
istream& read(char*buffer,streamsize size);
</pre>
