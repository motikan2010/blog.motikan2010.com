　関数inputで入力された値は文字列型として扱われる。  
「16進数 → 10進数 → バイナリデータ」の順で変換しています。

<div class="md-code">
```python
import struct

def main():
  str = input()

  str_list = str.split(" ")

  f = open('test.dat', 'wb')
  for hex in str_list:
    f.write(struct.pack('B', int(hex, 16)))

  f.close()

if __name__ == "__main__":
    main()

```
</div>

ここでは「hex_sample.py」で保存

<div class="md-code">
```bash
# python hex_sample.py
68 65 6c 6c 6f 20 77 6f 72 6c 64 21 21

# hexdump -C test.dat
00000000  68 65 6c 6c 6f 20 77 6f  72 6c 64 21 21           |hello world!!|
0000000d
```
</div>  

　バイナリデータで保存されている。  
実用には例外処理なども記述するように。