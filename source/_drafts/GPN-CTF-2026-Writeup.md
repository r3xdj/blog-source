---
title: GPN CTF 2026 Writeup
tags:
  - forensics
  - reverse
  - misc
  - CTF
categories:
  - Cyber
  - CTF Writeup
description:
cover: goulash.png
top_img: cover.png
---


因為考上台大了，這個暑假打算上CTFtime上找一些比賽來打。本來想說畢業當周周末來打這個GPN CTF 2026，結果開賽前突然看到零日餅乾社裡傳一個ISIP-HS CTF，比賽時間包含這個GPN CTF，結果就變成GPN CTF只解了兩三題(還簡單題XD)(都在打ISIP-HS CTF)。

# Sanity Check
![Sanity Check](2026-06-08-17-35-40.png)
Welcome題，點進Rule看即可
![rule](2026-06-08-17-38-00.png)
Flag: `GPNCTF{Yes Chef! I am ready for a fair competition}`

# Double Fried
![Double Fried](2026-06-08-17-38-52.png)
題目給了一個pcap檔案，用Wireshark去開
![](2026-06-08-17-48-48.png)
![](2026-06-08-17-51-19.png)
發現訊息有R開頭跟F開頭兩種，而從圖一可以猜出F開頭的是雜訊，所以我們只看R開頭的。
用`frame contains "R"`進行篩選
然後手動把Flag字母接起來~~因為我也只會人工接起來~~
Flag: `GPNCTF{NICe, yOu FoUnD 0U7 wHO dID NOt b3L0ng ThEr3}`

# Auto Cooker
![Auto Cooker](2026-06-08-17-43-39.png)
把題目給的檔案丟進IDA逆向
```c
int __fastcall main(int argc, const char **argv, const char **envp)
{
  _BOOL4 v4; // [rsp+1Ch] [rbp-4h]

  v4 = 0;
  if ( argc > 1 )
    v4 = strcmp(argv[1], "cooking_class") == 0;
  printf("%s", HEADER[0]);
  printf("%s\n\n", WELCOME);
  puts("Enter your recipe (flag) you want to cook and confirm with [ENTER]:");
  fgets(RECIPE, 64, _bss_start);
  check_recipe_length();
  FOOD = *(_QWORD *)RECIPE;
  qword_404128 = qword_4040E8;
  qword_404130 = qword_4040F0;
  qword_404138 = qword_4040F8;
  qword_404140 = qword_404100;
  qword_404148 = qword_404108;
  qword_404150 = qword_404110;
  qword_404158 = qword_404118;
  explain_current_food(v4);
  salt();
  explain_current_food(v4);
  fry();
  explain_current_food(v4);
  trim();
  explain_current_food(v4);
  mix();
  explain_current_food(v4);
  taste();
  puts("Congratulations, you \"cooked\" a delicious plate of food!");
  return 0;
}
```
發現它會對輸入做一連串處理，最後`taste`品嘗一下
看起來是他會檢查你的輸入是不是Flag，是的話就會Congratulations
首先是我們後面的函式如果要執行，都需要`v4`為真才行(看完下面程式碼你就動了)，所以我們在執行程式時需要傳入`cooking_class`這個參數。也就是

```bash
./autocooker cooking_class
```

接著一一分析每個函式

## check_recipe_length

```c
__int64 check_recipe_length()
{
  __int64 result; // rax

  if ( RECIPE[TARGET_LENGTH] || (result = (unsigned __int8)RECIPE[TARGET_LENGTH - 1], !(_BYTE)result) )
  {
    puts("Your recipe is too complicated or too simple, I already know it won't taste good :(");
    exit(1);
  }
  return result;
}
```

我們點進去`TARGET_LENGTH`可以看到他的數值為`3Dh`，轉成10進位就是61

## salt
```c
__int64 salt()
{
  __int64 result; // rax
  unsigned int i; // [rsp+Ch] [rbp-4h]

  puts("Salting...");
  for ( i = 0; ; ++i )
  {
    result = i;
    if ( i > 0x3F )
      break;
    *((_BYTE *)&FOOD + (int)i) ^= GRAIN_OF_SALT;
  }
  return result;
}
```

GRAIN_OF_SALT = 0AAh
salt()：將每個字都和0xAA XOR


## fry

```c
__int64 fry()
{
  __int64 result; // rax
  signed int i; // [rsp+Ch] [rbp-4h]

  puts("Frying...");
  for ( i = 0; ; ++i )
  {
    result = (unsigned int)i;
    if ( (unsigned int)i > 0x3F )
      break;
    *((_BYTE *)&FOOD + i) = (*((_BYTE *)&FOOD + i) >> 4) | (16 * *((_BYTE *)&FOOD + i));
  }
  return result;
}
```

fry()：交換高低位
如0xED變成0xDE

## trim

```c
__int64 trim()
{
  __int64 result; // rax
  unsigned int i; // [rsp+Ch] [rbp-4h]

  puts("Oops, it burned :(\nCutting off the burnt bits...");
  for ( i = TARGET_LENGTH; ; ++i )
  {
    result = i;
    if ( i > 0x3F )
      break;
    *((_BYTE *)&FOOD + (int)i) &= 0xFu;
  }
  return result;
}
```

trim()：刪掉末三個數的高位 
0xAA 0xAA 0xAA --> 0x0A 0x0A 0x0A

## mix

```c
__int64 mix()
{
  __int64 result; // rax
  _QWORD v1[9]; // [rsp+0h] [rbp-50h]
  unsigned int i; // [rsp+4Ch] [rbp-4h]

  puts("Mixing...");
  v1[0] = FOOD;
  v1[1] = qword_404128;
  v1[2] = qword_404130;
  v1[3] = qword_404138;
  v1[4] = qword_404140;
  v1[5] = qword_404148;
  v1[6] = qword_404150;
  v1[7] = qword_404158;
  for ( i = 0; ; ++i )
  {
    result = i;
    if ( i > 0x3F )
      break;
    *((_BYTE *)&FOOD + (int)i) = *((_BYTE *)v1 + 63LL - (int)i);
  }
  return result;
}
```

mix()：整串顛倒

## taste

```c
__int64 taste()
{
  __int64 result; // rax
  unsigned int i; // [rsp+8h] [rbp-8h]
  int v2; // [rsp+Ch] [rbp-4h]

  puts("Taste testing...");
  v2 = 0;
  for ( i = 0; ; ++i )
  {
    result = i;
    if ( i > 0x3F )
      break;
    v2 |= *((_BYTE *)&FOOD + (int)i) != (unsigned __int8)DELICIOUS[i];
  }
  if ( v2 )
  {
    puts("YUCK!\nOur taste tester thinks your recipe produces bad food, we cannot serve this...");
    exit(1);
  }
  return result;
}
```

我們只要把四個步驟逆著推回去就好

## exploit
```python
#!/usr/bin/python3

delicious = [
    0x0A, 0x0A, 0x0A, 0x0A, 0x7D, 0xDD, 0x5E, 0x4E,
    0x5F, 0x9D, 0xFC, 0x2C, 0x9D, 0xB9, 0xEC, 0x5F,
    0xEF, 0xF9, 0x99, 0xEE, 0x8F, 0xE9, 0x2E, 0x5F,
    0x8D, 0xFF, 0x5C, 0x5F, 0x8F, 0x5E, 0xCC, 0x5F,
    0x3D, 0xEE, 0xE9, 0xFE, 0x8F, 0x5F, 0xFE, 0x8F,
    0xE9, 0x5F, 0xFD, 0xA9, 0x3F, 0x5F, 0x99, 0x1E,
    0x3E, 0x6C, 0x5F, 0x6C, 0xFC, 0xFC, 0xCE, 0x5F,
    0x3E, 0x1D, 0xCE, 0xEF, 0x9E, 0x4E, 0xAF, 0xDE
]

delicious.reverse() # re-mix
# re-trim
for i in (61,62,63):
    delicious[i] = 0xAA
delicious[60] = 0x0A
# re-fry
for i in range(64):
    delicious[i] = (delicious[i] >> 4 | delicious[i] << 4) & 0xFF
# re-salt
for i in range(64):
    delicious[i] ^= 0xAA

print("".join( chr(h) for h in delicious))
```