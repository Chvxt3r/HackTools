# Useful Linux Commands

### Virtual Environments
```bash
virtualenv <env name>
source <env name>/bin/activate
# install whatever using pip
deactivate
```
### Git
Store credentials
```bash
git config --global credential.helper store 
# Then do a push, enter username, PAT as password.
```
Add File
```bash
git add <file>
```
Add All Files
```bash
git add .
git add --all
```
Commit Changes
```bash
git commit -m "<commit message>"
```
Push to github
```bash
git push
```
Update repository URI
```bash
git remote set-url origin https://github.com/username/reponame.git
```
