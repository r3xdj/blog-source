---
title: '[Note] Writing a simple Program in C - LiveOverflow'
tags:
  - C
  - vim
categories:
  - Cyber
description: 看了LiveOverflow 的 Writing a simple Program in C 這部影片之後做的簡要筆記
cover: output.png
top_img: output.png
date: 2026-06-03 12:09:45
updated: 2026-06-03 12:09:45
---


看了這部影片 [Writing a simple Program in C - YouTube](https://youtu.be/JGoUaCmMNpE?list=PLhixgUqwRTjxglIswKp9mpkfPNfHkzyeN) 做的筆記

# PATH

```bash
env | grep "PATH" # or
echo $PATH #this one

# add to PATH(for one time)
export PATH=$PATH:directory
```

# vim 基礎

```bash
:syntax on #上色
:set number #標號
```

# C 語言

```c
#include <stdio.h>
int main(int argc, char *argv[]){
    if(argc==2){
        printf("Knock, Knock, %s\n", argv[1]);
    } else {
        fprintf(stderr, "Usage: %s <name>\n", argv[0]);
        return 1;
    }
}
```

解析：

- argc: 參數數量(從檔案名稱算起)
- argv: 參數矩陣(包含檔案名稱)
  範例：
  
  ```c
  #include <stdio.h>
  int main(int argc, char *argv[]){
    printf("argc: %d\n",argc);
    printf("argv:\n")
    for(int i=0;i<argc;i++){
        printf("\targv[%d]: %s\n", i, argv[i]);
    }
    return 0;
  }
  ```
  
  輸出：
  ![output](output.png)
