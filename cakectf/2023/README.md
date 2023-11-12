# CAKE CTF 2023
## NANDE

Giải thích một chút về hàm `main`:
```C
int __cdecl main(int argc, const char **argv, const char **envp)
{
  char v4; // [rsp+20h] [rbp-38h]
  unsigned __int64 j; // [rsp+28h] [rbp-30h]
  unsigned __int64 i; // [rsp+30h] [rbp-28h]
  unsigned __int64 k; // [rsp+38h] [rbp-20h]
  char *Str; // [rsp+40h] [rbp-18h]

  if ( argc >= 2 )
  {
    Str = (char *)argv[1];
    if ( strlen(Str) != 32 )
      goto $wrong;
    for ( i = 0; i < 0x20; ++i )
    {
      for ( j = 0; j < 8; ++j )
        InputSequence[8 * i + j] = (Str[i] >> j) & 1;
    }
    CIRCUIT(InputSequence, OutputSequence);
    v4 = 1;
    for ( k = 0; k < 0x100; ++k )
      v4 &= OutputSequence[k] == AnswerSequence[k];
    if ( v4 )                                
    {
      puts(string);                         // Correct!
      return 0;
    }
    else
    {
$wrong:
      puts(aWrong);
      return 1;
    }
  }
  else
  {
    printf("Usage: %s <flag>\n", *argv);
    return 1;
  }
}
```

Đại khái thì mình sẽ chạy chương trình với tham số là chuỗi flag có độ dài 32 bytes. 

Mảng InputSequence sẽ lưu lại từng bit của Flag (len(InputSequence) = 32 * 8 = 256). InputSequence sẽ đi qua hàm CIRCUIT và trả về mảng OutputSequence. 

Trong hàm CIRCUIT có gọi hàm MODULE. Rút gọn biểu thức logic của cái đống NAND thì mình sẽ có được MODULE(a, b) = a'b * ab' = XOR(a, b). 
```C
void __fastcall MODULE(unsigned __int8 a, unsigned __int8 b, bool *x)
{
  unsigned __int8 z; // [rsp+20h] [rbp-18h] BYREF
  unsigned __int8 v4; // [rsp+21h] [rbp-17h] BYREF
  char v5[6]; // [rsp+22h] [rbp-16h] BYREF

  NAND(a, b, (bool *)&z);                       // z = (a nand b)
  NAND(a, z, (bool *)v5);                       // v5 = (a nand z)
  NAND(b, z, (bool *)&v4);                      // v4 = (b nand z)
  NAND(v5[0], v4, x);                           // x = v5 nand v4
}
```

Giờ thì rev lại hàm CIRCUIT là được :|

```python
out = [1, 1, 1, 1, 1, 0, 0, 1, 1, 0, 0, 1, 0, 0, 1, 0, 0, 1, 1, 0, 0, 0, 0, 1, 1, 1, 1, 1, 0, 0, 1, 1, 1, 1, 0, 1, 0, 1, 1, 1, 0, 0, 0, 1, 1, 0, 1, 1, 1, 1, 0, 1, 0, 1, 0, 0, 1, 0, 1, 0, 1, 1, 0, 1, 0, 0, 1, 1, 0, 1, 1, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 1, 0, 0, 1, 0, 0, 1, 0, 1, 1, 1, 0, 0, 1, 1, 1, 0, 0, 1, 1, 1, 0, 1, 0, 1, 1, 1, 1, 0, 1, 1, 0, 0, 0, 1, 1, 0, 0, 1, 1, 0, 1, 1, 0, 0, 0, 1, 0, 1, 1, 1, 0, 1, 0, 0, 0, 0, 1, 0, 0, 0, 0, 1, 1, 1, 0, 1, 0, 0, 0, 1, 1, 0, 0, 1, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0, 1, 0, 1, 0, 0, 1, 1, 0, 1, 1, 1, 0, 1, 0, 1, 0, 1, 0, 0, 0, 0, 0, 0, 1, 1, 0, 1, 1, 0, 1, 1, 0, 1, 0, 1, 0, 1, 0, 0, 1, 1, 1, 1, 1, 0, 1, 1, 0, 1, 1, 1, 0, 0, 1, 0, 1, 1, 0, 1, 1, 0, 0, 1, 0, 0, 1, 1, 0, 0, 1, 1, 1, 0, 1, 0, 1, 0, 1, 1, 0] # Lấy từ mảng AnswerSequence.

inp = [0] * 256

for i in range (0, 4660):
    inp[255] = out[255] ^ 1
    for j in range (254, -1, -1):
        inp[j] = out[j] ^ inp[j + 1]
    out = inp

flag = ""
for i in range (0, 255, 8):
    bin = ""
    for j in range (i, i + 8):
        bin = str(inp[j]) + bin
    flag = flag + chr(int(bin, 2))  

print(flag)
```

## CAKEPUZZLE

Bài này thì nó hơi hơi meo meo. 
Xem thêm: https://en.wikipedia.org/wiki/15_Puzzle


Mình sẽ nhập vào một chuỗi, chương trình sẽ lấy chữ cái đầu trong cái chuỗi mình nhập. 

Mình đổi cái tên hàm cho nó dễ nhìn chút. Vòng lặp nó lặp hoài cho tới khi hàm Check_Table trả về 0.

```C
int __cdecl __noreturn main(int argc, const char **argv, const char **envp)
{
  char v3; // [rsp+0h] [rbp-70h]

  alarm(0x3E8u);
  while ( (unsigned int)Check_Table() )
  {
    printf("> ");
    fflush(_bss_start);
    if ( (unsigned int)__isoc99_scanf("%s") == -1 )
      exit(0);
    Swap_Data_Follow_Char(v3);
  }
  win();
}
```

Tóm tắt lại thì điều kiện dừng của vòng lặp là M[i][j] < M[i + 1][j] && M[i][j] < M[i][j + 1](cái này mình tưởng tượng M nó là mảng 2 chiều 4x4 thì đây là một mảng 2 chiều tăng dần từ trái sang phải, từ trên xuống dưới)

```C
__int64 Check_Table()
{
  int j; // [rsp+0h] [rbp-8h]
  int i; // [rsp+4h] [rbp-4h]

  for ( i = 0; i <= 2; ++i )
  {
    for ( j = 0; j <= 2; ++j )
    {
      if ( M[4 * i + j] >= M[4 * i + 1 + j] )
        return 1LL;
      if ( M[4 * i + j] >= M[4 * i + 4 + j] )
        return 1LL;
    }
  }
  return 0LL;
}
```

Tiếp theo mình xem thử cái hàm này.

```C
__int64 __fastcall Swap_Data_Follow_Char(char a1)
{
  __int64 result; // rax
  unsigned int v2; // [rsp+10h] [rbp-8h] BYREF
  unsigned int v3; // [rsp+14h] [rbp-4h] BYREF

  get_pos_value_0((int *)&v3, (int *)&v2);      // Tọa độ mà M[i][j] = 0
  result = (unsigned int)a1;
  if ( (_DWORD)result == 'U' )
  {
    result = v3;
    if ( v3 )
      return (__int64)Swap(&M[4 * v3 + v2], &M[4 * (v3 - 1) + v2]);
  }
  else if ( (int)result <= 'U' )
  {
    if ( (_DWORD)result == 'R' )
    {
      result = v2;
      if ( v2 != 3 )
        return (__int64)Swap(&M[4 * v3 + v2], &M[4 * v3 + v2 + 1]);
    }
    else if ( (int)result <= 'R' )
    {
      if ( (_DWORD)result == 'D' )
      {
        result = v3;
        if ( v3 != 3 )
          return (__int64)Swap(&M[4 * v3 + v2], &M[4 * (v3 + 1) + v2]);
      }
      else if ( (_DWORD)result == 'L' )
      {
        result = v2;
        if ( v2 )
          return (__int64)Swap(&M[4 * v3 + v2], &M[4 * v3 + v2 - 1]);
      }
    }
  }
  return result;
}
```

Hàm này check 4 case `'U', 'D', 'L', 'R'` rồi swap có giá trị 0 với 1 trong 4 ô kế nó.

Cái bài này nó hơi khác với cái game bth, các giá trị nó xếp theo kiểu:
```
0   1   2   3
4   5   6   7
8   9   10  11
12  13  14  15
```
nên là mình cũng chả kiếm được cái tool để chơi theo kiểu này. :( Đành phải tự mò thôi :v

Code lại cái quả này để check cho dễ :V
```python
# gamelor.py
# Chuyển giá trị của mảng M thành các giá trị từ 0->15 cho dễ xử lí.

state = [
    [5, 6, 1, 10],
    [12, 15, 13, 9],
    [0, 11, 4, 7],
    [2, 8, 3, 14]
]

def get_pos():
    for i in range (4):
        for j in range (4):
            if (state[i][j] == 0):
                return i, j
    return -1, -1

def print_state():
    for r in state:
        print(r)

def go(c):
    i, j = get_pos()
    if (c == 'U' and i != 0):
        state[i][j], state[i - 1][j] = state[i - 1][j], state[i][j]
    elif (c == 'D' and i != 3):
        state[i][j], state[i + 1][j] = state[i + 1][j], state[i][j]
    elif (c == 'L' and j != 0):
        state[i][j], state[i][j - 1] = state[i][j - 1], state[i][j]
    elif (c == 'R' and j != 3):
        state[i][j], state[i][j + 1] = state[i][j + 1], state[i][j]
    else:
        print("Sai")
while True:
    print_state()
    c = input("> ")
    go(c)

# Goal state:
# 0   1   2   3
# 4   5   6   7
# 8   9   10  11
# 12  13  14  15
```
