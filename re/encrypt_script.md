## TEA相关算法

### TEA(Tiny Encryption Algorithm)

- 明文（块）/密文（块）长度：64比特
- 密钥长度：128比特
- 建议迭代次数：32

```python
#加密算法
def encrypt(v, k):
    delta = 0x9e3779b9
    
    v0 = v[0]
    v1 = v[1]
    x = 0
    
    for i in range(32):
        x += delta
        x = x & 0xffffffff
        #考虑寄存器溢出
        v0 += ((v1 << 4) + k[0]) ^ (v1 + x) ^ ((v1 >> 5) + k[1])
        v0 = v0 & 0xffffffff
        v1 += ((v0 << 4) + k[2]) ^ (v0 + x) ^ ((v0 >> 5) + k[3])
        v1 = v1 & 0xffffffff
        
    v[0] = v0
    v[1] = v1
    return v


#解密算法
def decrypt(v, k):
    delta = 0x9e3779b9
    
    v0 = v[0]
    v1 = v[1]
    x = 0xc6ef3720
    #(0x9e3779b9 * 32) & 0xffffffff
    
    for i in range(32):
        v1 -= ((v0 << 4) + k[2]) ^ (v0 + x) ^ ((v0 >> 5) + k[3])
        v1 = v1 & 0xffffffff
        v0 += ((v1 << 4) + k[0]) ^ (v1 + x) ^ ((v1 >> 5) + k[1])
        v0 = v0 & 0xffffffff
        x -= delta
        x = x & 0xffffffff
        
    v[0] = v0
    v[1] = v1
    return v


plaintext = [v0, v1]
#每一部分为32位无符号整数
key = [k0, k1, k2, k3]

cyphertext = encrypt(plaintext, key)
message = decrypt(cyphertext, key)
#message == plaintext
```



### XTEA

```python
#加密算法
def encrypt(rounds, v, k):
    delta = 0x9e3779b9
    
    v0 = v[0]
    v1 = v[1]
    x = 0
    
    for i in range(rounds):
        v0 += (((v1 << 4) ^ (v1 >> 5)) + v1) ^ (x + k[x & 3])
        v0 = v0 & 0xffffffff
        x += delta
        x = x & 0xffffffff
        v1 += (((v0 << 4) ^ (v0 >> 5)) + v0) ^ (x + k[(x >> 11) & 3])
        v1 = v1 & 0xffffffff
        
    v[0] = v0
    v[1] = v1
    return v


#解密算法
def decrypt(v, k):
    delta = 0x9e3779b9
    
    v0 = v[0]
    v1 = v[1]
    x = 0xc6ef3720
    
    for i in range(32):
        v1 -= (((v0 << 4) ^ (v0 >> 5)) + v0) ^ (x + k[(x >> 11) & 3])
        v1 = v1 & 0xffffffff
        x -= delta
        x = x & 0xffffffff
        v0 -= (((v1 << 4) ^ (v1 >> 5)) + v1) ^ (x + k[x & 3])
        v0 = v0 & 0xffffffff
        
    v[0] = v0
    v[1] = v1
    return v


plaintext = [v0, v1]
key = [k0, k1, k2, k3]
rounds = 32

cyphertext = encrypt(rounds, plaintext, key)
message = decrypt(rounds, cyphertext, key)
```



### XXTEA(Corrected Block TEA)

```python
def shift(z, y, x, k, p, e):
    return ((((z >> 5) ^ (y << 2)) + ((y >> 3) ^ (z << 4))) ^ ((x ^ y) + (k[(p & 3) ^ e] ^ z)))


#加密算法
def encrypt(v, k):
    n = len(v)
    rounds = 6 + 52 // n
    delta = 0x9e3779b9
    
    x = 0
    z = v[n - 1]
    
    for i in range(rounds):
        x += delta
        x = x & 0xffffffff
        e = (x >> 2) & 3
        for p in range(n - 1):
            y = v[p + 1]
            v[p] = v[p] + shift(z, y, x, k, p, e)
            v[p] = v[p] & 0xffffffff
            z = v[p]
        
        p += 1
        y = v[0]
        v[n - 1] = (v[n - 1] + shift(z, y, x, k, p, e))
        v[n - 1] = v[n - 1] & 0xffffffff
        z = v[n - 1]
        
    return v


#解密算法
def decrypt(v, k):
    n = len(v)
    rounds = 6 + 52 // n
    delta = 0x9e3779b9
    
    x = (rounds * delta) & 0xffffffff
    y = v[0]
    
    for i in range(rounds):
        e = (x >> 2) & 3
        for p in range(n - 1, 0, -1):
            z = v[p - 1]
            v[p] = v[p] - shift(z, y, x, k, p, e)
            v[p] = v[p] & 0xffffffff
            y = v[p]
    
        p = 0
        z = v[n - 1]
        v[0] = v[0] - shift(z, y, x, k, p, e)
        v[0] = v[0] & 0xffffffff
        y = v[0]
        x -= delta
        x = x & 0xffffffff
        
    return v
        

plaintext = [v0, v1]
key = [k0, k1, k2, k3]

cyphertext = encrypt(plaintext, key)
message = decrypt(cyphertext, key)      
```





## RC4

```python
def swap(a, b):
    t = a
    a = b
    b = t
    return a, b

def encrypt(K, m):
    S = [0 for i in range(256)]
    T = [0 for i in range(256)]
    k = [0 for i in range(len(m))]

    for i in range(256):
        S[i] = i
        T[i] = K[i % len(K)]
    
    j = 0
    for i in range(256):
        j = (j + S[i] + T[i]) % 256
        S[i], S[j] = swap(S[i], S[j])

    i = 0
    j = 0
    for x in range(len(m)):
        i = (i + 1) % 256
        j = (j + S[i]) % 256
        S[i], S[j] = swap(S[i], S[j])
        t = (S[i] + S[j]) % 256
        k[x] = S[i]
```





## ROT

```python
#right shift n
r_shift = n
s = 'XXXXXXXxxxxx1111'
res = ''
for i in s:
    if(i >= 'A' and i <= 'Z'):
        res += chr((ord(i) - 65 + r_shift) % 26 + 65)
    elif(i >= 'a' and i <= 'z'):
        res += chr((ord(i) - 97 + r_shift) % 26 + 97)
    else:
        res += i
print(res)
```





## Base

### Base64换表

```python
import base64

str1 = "VDomYv=="

des = "ZABCDEFGHIJKLMNOPQRSTUVWXYzabcdefghijklmnopqrstuvwxy0123456789+/"
src = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/"

maptab = str.maketrans(des,src)
print(base64.b64decode(str1.translate(maptab)))
```



### Base64特征

```python
a[0] = s[0] >> 2
#a[0] = s[0] // 4

a[1] = (s[0] & 3) << 4 + (s[1] >> 4)
#a[1] = (s[0] % 4) * 16 + (s[1] // 16)

a[2] = (s[1] & 0xf) << 2 + (s[2] >> 6)
#a[2] = (s[1] % 16) * 4 + (s[2] // 64)

a[3] = s[2] & 0x3f
#a[3] = s[2] % 64
```



​    

