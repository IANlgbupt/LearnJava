#git解决


----------

 - 遇到以下问题，每次用git将本地文件上传到远程仓库时，都要输入uername和password.
    解决方法：
        
先 git config --system --unset credential.helper
然后 git config --global credential.helper store
     git pull / git push （第一次输入需要，后续不需要）