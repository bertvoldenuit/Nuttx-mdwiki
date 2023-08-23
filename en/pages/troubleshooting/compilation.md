# Compilation



This exercise is based on ```Real-time Operating Systems Book 2 - The Practice, by Jim Cooling``` chapter 2 - exercise 1

## unsupporte character
-----------------------

During compilation with `make menuconfig` the following error appear: error: ` warning: ignoring unsupported character '`

The problem might come from windows vs unix file format. To convert all file to unix format:

```
cd ~/nuttxspace/nuttx
sudo apt-get install dos2unix
apt-get install dos2unix
```

or you can try git reset:

```
git fetch
git reset --hard origin/master
or
git reset --hard origin/<branch_name>
```

Explanation:

`git fetch` downloads the latest from remote without trying to merge or rebase anything.

Then the `git reset` resets the master branch to what you just fetched. The `--hard` option changes all the files in your working tree to match the files in origin/master