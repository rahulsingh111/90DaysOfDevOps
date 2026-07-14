# Git commands reference

```bash
git --version

git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
git config --list

mkdir devops-git-practice && cd devops-git-practice
git init
git status
ls -a .git/

touch git-commands.md

git add git-commands.md
git status
git commit -m "Add initial Git commands reference"
git log

git status
git add git-commands.md && git commit -m "Add more Git commands to reference"

git log --oneline
```