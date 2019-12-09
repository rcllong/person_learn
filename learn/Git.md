## 把本地文件夹上传到github
* 把文件夹变成Git可管理的仓库
```git
git init .
```

* 将所有文件提交到暂存区
```git
git add .
```

* 将所有文件提交
```git
git commit -m 'Commit all file'
```
* 在github上初始化git仓库
![git仓库](https://raw.githubusercontent.com/rcllong/person_learn/master/%E9%99%84%E4%BB%B6/git%E4%BB%93%E5%BA%93.png)

* 把本地仓库和git仓库绑定
```git
git remote add origin https://github.com/rcllong/SpringCloudLearning.git
```

* 把本地文件和git仓库文件合并
```git
git pull --rebase origin master
```

* 把本地文件推送
```git
git push -u origin master
```



