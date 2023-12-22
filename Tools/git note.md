## change account
`git config --global user.email "your email"`

## cache
`git add <file>`

## commit
`git commit -m "your comment"`

## push
`git push -?`
`git push <remote_name> <branch_name>`

## pull
`git pull <remote_name> <branch_name>`

### Remote control
Check the connected remote repository:  
`git remote -v`

Change the url of an existing remote repository:  
`git remote set-url <remote_name> <new_repository_URL>`  
`git remote set-url --push origin <url>`  

Add a new remote repository:  
`git remote add <remote_name> <repository_URL>`

## 登出
`git credential-cache exit`
