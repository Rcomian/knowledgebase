[Home](../index.md)

## Bash tips

![bash splash](res/bash-splash.png)

### Replace text in a file with a unix style path

```bash
find='@PLACEHOLDER@'
replace=`pwd` #replace is the text to be inserted

sed -i "s/${PLACEHOLDER}/${cwd//\//\\\/}/g" targetfile.conf
```
