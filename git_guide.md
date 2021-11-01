#git最基础的使用
---
<pre>
cd  D:/git
mkdir book
cd book

git init（进入book下再初始化否则就会变成book/book)
git remote add origin git@github.com:pipsqueakliu/book.git添加远程仓库


git add README.md
git status
git commit -m "explain"


git push -u origin master 将本地当前分支内容推从到github的book的master分支中
git push -u origin feature-D 将本地当前分支内容推送到github的book的feature-D分支中

git clone git@github.com:pipsqueakliu/book.git
将book克隆到本地当前文件夹

</pre>

<pre>
已提交到github去的内容如何删除
###如果不存在的话（存在的话就不用初始化和建立连接了）###
git init（进入book下再初始化否则就会变成book/book)
git remote add origin git@github.com:pipsqueakliu/book.git添加远程仓库

##通过连接将项目pull下来####
git pull origin master       # 将远程仓库里面的项目拉下来

ls -a           # 查看有哪些文件夹

git rm -r --cached .idea # 删除.idea文件夹
git commit -m '删除.idea' # 提交,添加操作说明

git push -u origin master # 将本次更改更新到github项目上去
</pre>


#git高阶使用
