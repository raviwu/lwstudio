+++
title = "終端機工具"
date = 2016-03-27T04:07:31+08:00
publishdate = 2016-03-27T04:07:31+08:00
tags = ["shell", "coding"]
draft = false

description = ""
summary = ""
keywords = []

[amp]
    elements = []

[author]
    name = "Ravi Wu"
    homepage = "https://raviwu.github.io"

[image]
    src = ""

[sitemap]
    changefreq = "monthly"
    priority = 0.5
    filename = "sitemap.xml"
+++

轉型後端工程師的路上，看了很多教學，裝了很多套件，用過很多軟體。目前用得還算不手殘的工具只有終端機（terminal）跟文本編輯器（text editor）。

因為要經常性切換不同的 Git Branch，MacOX 本來附掛的 terminal 顯得有點陽春，需要極大注意力才能清楚知道自己究竟目前身在何處又在哪支 branch 裡。

先烈做了很多很棒的工具改善工作體驗與視覺效果，iTerm 2 很是不錯，搭配上一些快捷熱鍵可以有效增加效率。

## Shortcut of iTerms2

### windows and tabs

`command + T` to open new tab

`shift + command + [ / ]` to switch between tabs

`command + W` to close the tab

`command + D` to have vertical divided windows

`command + [ / ]` to switch between windows

### cursor

`ctrl + a` to go to the beginning of line

`ctrl + e` to go to the end of line

`ctrl + f/b` to move the cursor

`ctrl + p` to get last command

`ctrl + r` to search history

`ctrl + d` to delete the current character

`ctrl + h` to delete previous characters from current cursor

`ctrl + w` to delete previous word from current cursor

`ctrl + k` to delete following charaters from current cursor

`command + /` to locate cursor position

### font size

`command + -/+` to decrease or increase the font size

## Shell

除此之外，還從預設的 [bash shell（bash）轉成 z shell（zsh](http://icarus4.logdown.com/posts/177661-from-bash-to-zsh-setup-tips)，搭配華麗的 [oh-my-zsh](http://ohmyz.sh/) 主題 [agnoster](https://gist.github.com/3712874)，記得補安裝 [PowerLine Font](https://github.com/powerline/fonts) 以免亂碼，讓後端工作也可以帶點美感，使用 Homebrew 安裝很是方便。

工具貴在熟練不在多，閒暇之餘，從 [Learn Enough Command Line to be Dangerous](https://www.learnenough.com/command-line-tutorial) 開始訓練肌肉記憶，專精之路尚遠矣。

### See Usage of specific command: `man`

`man command` checks out the manual for the command, for instance use `man echo` to see the description of the `echo` command.

### Common quit from mass or manufal windows

Use `q` to quit the manual mode, normally `q` works for the similar mode entered by other command like `ri Array`.

Use `ctrl + C` to kill current exucution of code, break the unfinit loop, etc.

Use `echo "some string" > file.txt` to through `"some string"` to `"file.txt"`. use `echo "another thing" >> file.txt` to *concat* the `"another string"` into the `"file.txt"`. By default `echo` appends `\n` to the end of the string.

### Simple output of a file: `cat`

Output the file content on the command line use `cat` command.

`cat file_1.txt file_2.txt > combined_file.txt` can pass the `file_1.txt` and `file_2.txt` content to the `combined_file.txt`.

#### Inspecting a file

`head file_name.txt` list the first 10 lines of a file

`tail file_name.txt` list the last 10 lines of a file

`wc file_name.txt` list the "line counts", "word counts", and "bytes" of the file

`less file_name.txt` has `/search_word` function to inspect the file [less wiki page](https://en.wikipedia.org/wiki/Less_(Unix))

### View the files within directory: `ls`

`ls -l` list out files in long format

`ls -la` list out all files in long format, including the hidden files

`ls -rtl` list out files in long format, with reversed order by recent modify time.

`ls -lh` list out files in long format, with human readable file size count (K instead of bytes).

### File manipulation: `mv`, `rm`, `cp`, `diff`

`mv file_a.txt file_b.txt` rename `file_a.txt` to `file_b.txt`

`rm file_a.txt` remove `file_a.txt` `rm -rf file_a.txt` force remove `file_a.txt`

`cp file_a.txt file_b.txt` copy `file_a.txt` to `file_b.txt`

`diff file_a.txt file_b.txt` to show the difference between two file. There won't be output if the two files has no difference.

### Check if programme installed: `which`

`which ruby` checks whether ruby is installed in the computer

### user `grep` to catch the specified string in file

`grep -i string file.txt` means catch all case insensitive "string" in "file.txt"

`grep string file.txt | wc` pipes the result of `grep` and pass it to wordcount programme

`grep -ri string directory` can find string from the very deep level, starts from the directory, case insentitive

### managing process status with `ps`

`ps aux | grep string` in processes status shows in aux options and pipe result to grep then grep the "string" form ps aux result

`kill -15 <pid>` to kill the individual process with process id

`pkill -15 -f spring` can kill all process contains string spring

### directories

home directory is normally `/Users/username/` alias `~/`

use `sudo command` to get root permission for operation

`mkdir` to make new directory

`pwd` to print current working directory

`find . -name '*.txt'` to find files whose names match the pattern `*.txt`, starting in the current directory . and then in its subdirectories

`open file.ext` will open the file.ext with default programe to the .ext files

`open .` will then execute finder and open current directory

### combining commands in single line

`command line1 ; command line2 ; command line 3` use `;` to combine three commands in single line, and execute the three commands one after another

`command line1 && command line2 && command line3` works similar as `;` seperator, using `&&` can make the following command execute only if previous command successfully executed.

### Configure the shell

edit `~/.zshrc` file with the resired configs and save it

`source .zshrc` to reload the config

`.bashrc` is the config file for bash shell
