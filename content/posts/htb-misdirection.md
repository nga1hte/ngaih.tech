---
layout: post
title: "[HTB] misDIRection writeup"
slug: "misdirection"
date: "2022-12-31"
comments: false
categories:
  - hacking
tags:
  - ctf
  - write up
  - hack the box
---

This is my writeup for misDIRection, which is a miscellaneous challenge in hackthebox. The challenge seems fairly simple and involves a zip file that contains directories and files in a specific order. Upon inspection, the files are empty and probably why the title of the challenge is misdirection.

## Steps for finding the flag

Unzipping the file yields a directory with the name ```.secret```, which is hidden in unix based systems.

```
$ unzip misDIRection.zip -d misDIRection & ls -a misDIRection
. .. .secret
```

Checking the ```.secret``` directory we get a bunch of directories with empty file contents.

```
$ ls -a
0 2 4 6 8 a b c d e f g h i j k l m n o p q r s t u v w x y z
1 3 5 7 9 A B C D E F G H I J K L M N O P Q R S T U V W X Y Z
```

At this point we can use ```tree``` to display all the files and subdirectories.

```
tree
```

Checking the content of the files, we find that they are empty files. On another observation since the files are all numbered and distinct it can be assumed that they are keys and since the directory names are all alphanumeric they can be assumed as values and hence forming a {key, value} pair.
Sorting the file names and taking the directory names as values we might be find the flag to our challenge.

We use tree prune to list only directories that are not empty, thereby eliminating values without keys

```
$ tree --prune -afi
.
./0
./0/6
./1
...
```
We pipe the output of the tree and cut each row between “/” and take only the second and third values and put space beween them using the cut tool. Refer to the man pages of each tool to know more.

```
$ tree --prune -afi | cut -s -d "/" -f 2,3 --output-delimiter=" " | sort -k 2n
0
1
2
5
9
...
...
S 1
F 2
R 3
...
```

Since there are still some directories without keys lingering in our output, we will remove them using ```sed``` which will remove all rows without space.

```
$ tree --prune -afi | cut -s -d "/" -f 2,3 --output-delimiter=" " | sort -k 2n | sed '/ /!d'
S
F
R
C
...
```

For printing all the values, lets use ```tr``` a to remove newlines form all the rows.

```
$ tree --prune -afi | cut -s -d "/" -f 2,3 --output-delimiter=" " | sort -k 2n | sed '/ /!d' | cut -d " " -f 1 | tr -d '\n'
SFRCe0RJUjNjdEx5XzFuX1BsNDFuX1NpN2V9
```

The output values look fairly similar to base64 encoding, so let us pipe our output one more time to “base64” tool to decode it.

```
$ tree --prune -afi | cut -s -d "/" -f 2,3 --output-delimiter=" " | sort -k 2n | sed '/ /!d' | cut -d " " -f 1 | tr -d '\n' | base64 --decode
HTB{D********e}
```

> That's it! You have the flag.






