---
layout: default
title: Git - notes
categories: software
---
Squash the last three commits

```bash
git rebase -i HEAD~3
```

You will see something like this

```plain
pick f392171 Added new feature X
pick ba9dd9a Added new elements to page design
pick df71a27 Updated CSS for new elements
```

- `pick` `p` the commit will be taken
- `squash` `s`, the commit will be blended with the above.

Edit to something like this

```plain
pick f392171 New message for this three commit!
squash ba9dd9a
squash df71a27
```

Now you can accept the change by continuing the rebase:

```bash
git rebase --continue
```

After that you need to force override the history using this command:

```bash
git push origin master --force
```

## Squash merge

```bash
git checkout master
git merge --squash bugfix
git commit
```



### Checkout bare repository

```
git clone --bare https://github.com/nextcloud/server.git /var/www/nextcloud.git --recursive

cd nextcloud.git; git worktree add /var/www/nextcloud
cd /var/www/nextcloud; checkout -f v27.1.4

git --work-tree=/var/www/nextcloud --git dir=/home/artur/pub/drive/repos/nextcloud.git checkout -f v27.1.4 --recursive
```