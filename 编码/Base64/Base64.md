# Base64编码

## Base64基本概念 

> Base64编码：将字符串以每3个8比特(bit)的字节子序列拆分成4个6比特(bit)的字节(6比特有效字节，最左边两个永远为0，其实也是8比特的字节)子序列，再将得到的子序列查找Base64的编码索引表，得到对应的字符拼接成新的字符串的一种编码方式。

拆分过程示意图：

![划分前后](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d9405c1915a04343aa8624b06f523550~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

之所以取名叫做Base64编码是因为6bit的数一共有2的6次方（00000000-00111111）0-63一共64个数。

## Base64编码长度的变化


6和8的最大公倍数是24，因此，3个8bit字节刚好可以拆分成4个6bit字节，但是在计算机中一个字节需要8bit存储单元存储。所以将6bit前面两位补0。如下：

![补足前后](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e7090f22d2b74b41ab5116e11b90c4fc~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

补足后是32位，是原先所需要的24个的4/3倍。所以**Base64编码之后的长度是原先的4/3倍**

## Base64编码字符

Base64编码字符一共有64个，选用了`A-Z`、`a-z`、`0-9`、`+`、`/`64个可打印字符代表了64个二进制数。

    let base64EncodeChars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/'

## Base64编码举例

3字节编码
![3字节编码](https://img-blog.csdnimg.cn/7fc55fefa5b844ba843805199ffd078b.jpeg#pic_center)

1字节和2字节编码

![1字节和2字节编码](https://img-blog.csdnimg.cn/03128996362d49cbbda9c9e31b5472b6.png#pic_center)


注意：1字节和2字节的编码，如果后面位数不够则补0进行编码，后面两组没有数据，就用`=`补上。

## Base64编码原理

分析映射关系：abc -> xyzi。我们从高位到低位添加索引来分析这个过程

- x: (前面补两个0)a的前六位 => 00a7a6a5a4a3a2
- y: (前面补两个0)a的后两位 + b的前四位 => 00a1a0b7b6b5b4
- z: (前面补两个0)b的后四位 + c的前两位 => 00b3b2b1b0c7c6
- i: (前面补两个0)c的后六位 => 00c5c4c3c2c1c0

通过上述的映射关系，我们很容易得到下面的实现思路：


1. 将字符对应的ASCII编码转为8位二进制数
2. 将每三个8位二进制数进行以下操作

     - 将第一个数右移位2位，得到第一个6位有效位二进制数
     - 将第一个数 & 0x3之后左移位4位，得到第二个6位有效位二进制数的第一个和第二个有效位，将第二个数 & 0xf0之后右移位4位，得到第二个6位有效位二进制数的后四位有效位，两者取且得到第二个6位有效位二进制
     - 将第二个数 & 0xf之后左移位2位，得到第三个6位有效位二进制数的前四位有效位，将第三个数 & 0xC0之后右移位6位，得到第三个6位有效位二进制数的后两位有效位，两者取且得到第三个6位有效位二进制
     - 将第三个数 & 0x3f，得到第四个6位有效位二进制数

3. 将获得的6位有效位二进制数转十进制，查找对应Base64字符

C代码实现：

    // base64 转换表, 共64个
    static const char base64_alphabet[] = {
        'A', 'B', 'C', 'D', 'E', 'F', 'G',
        'H', 'I', 'J', 'K', 'L', 'M', 'N',
        'O', 'P', 'Q', 'R', 'S', 'T',
        'U', 'V', 'W', 'X', 'Y', 'Z',
        'a', 'b', 'c', 'd', 'e', 'f', 'g',
        'h', 'i', 'j', 'k', 'l', 'm', 'n',
        'o', 'p', 'q', 'r', 's', 't',
        'u', 'v', 'w', 'x', 'y', 'z',
        '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
        '+', '/'};

    static char cmove_bits(unsigned char src, unsigned lnum, unsigned rnum)
    {
        src <<= lnum; 
        src >>= rnum;
        return src;
    }
 
    int base64_encode(char *indata, int inlen, char *outdata, int *outlen)
    {
        int ret = 0;                                // 返回值
        if (indata == NULL || inlen == 0) {
            return ret = -1;
        }
    
        int in_len = 0;     // 源字符串长度, 如果in_len不是3的倍数, 那么需要补成3的倍数
        int pad_num = 0;    // 需要补齐的字符个数, 这样只有2, 1, 0(0的话不需要拼接)
        if (inlen % 3 != 0) 
        {
            pad_num = 3 - inlen % 3;
        }
        in_len = inlen + pad_num;       // 拼接后的长度, 实际编码需要的长度(3的倍数)

        int out_len = in_len * 8 / 6;   // 编码后的长度

        char *p = outdata;              // 定义指针指向传出data的首地址

        //编码, 长度为调整后的长度, 3字节一组
        for (int i = 0; i < in_len; i+=3) 
        {
            int value = *indata >> 2; // 将indata第一个字符向右移动2bit(丢弃2bit)
            char c = base64_alphabet[value]; // 对应base64转换表的字符
            *p = c; // 将对应字符(编码后字符)赋值给outdata第一字节

            //处理最后一组(最后3字节)的数据
            if (i == inlen + pad_num - 3 && pad_num != 0)
            {
                if(pad_num == 1) 
                {
                    *(p + 1) = base64_alphabet[(int)(cmove_bits(*indata, 6, 2) + cmove_bits(*(indata + 1), 0, 4))];
                    *(p + 2) = base64_alphabet[(int)cmove_bits(*(indata + 1), 4, 2)];
                    *(p + 3) = '=';
                } 
                else if (pad_num == 2)              // 编码后的数据要补两个 '='
                { 
                    *(p + 1) = base64_alphabet[(int)cmove_bits(*indata, 6, 2)];
                    *(p + 2) = '=';
                    *(p + 3) = '=';
                }
            }
            else                                // 处理正常的3字节的数据
            { 
                *(p + 1) = base64_alphabet[cmove_bits(*indata, 6, 2) + cmove_bits(*(indata + 1), 0, 4)];
                *(p + 2) = base64_alphabet[cmove_bits(*(indata + 1), 4, 2) + cmove_bits(*(indata + 2), 0, 6)];
                *(p + 3) = base64_alphabet[*(indata + 2) & 0x3f];
            }
        
            p += 4;
            indata += 3;
        }
    
        if(outlen != NULL) {
            *outlen = out_len;
        }
    
        return ret;
    }

需要注意的点是，在对一串数据分段进行Base64加密时，如果传入的二进制数据的原始长度不为3的倍数，那么最后得到的编码会添加`=`作为填充字符。以确保输出的长度是4的倍数。在解码时，这些填充字符会被忽略。这样，将会导致解码失败。

## Base64解码原理

分析映射关系 xyzi -> abc

- a: x后六位 + y第三、四位 => x5x4x3x2x1x0y5y4
- b: y后四位 + z第三、四、五、六位 => y3y2y1y0z5z4z3z2
- c: z后两位 + i后六位 => z1z0i5i4i3i2i1i0


1. 将字符对应的base64字符集的索引转为6位有效位二进制数
2. 将每四个6位有效位二进制数进行以下操作

   - 第一个二进制数左移位2位，得到新二进制数的前6位，第二个二进制数 & 0x30之后右移位4位，取或集得到第一个新二进制数
   - 第二个二进制数 & 0xf之后左移位4位，第三个二进制数 & 0x3c之后右移位2位，取或集得到第二个新二进制数
   - 第二个二进制数 & 0x3之后左移位6位，与第四个二进制数取或集得到第二个新二进制数
3. 根据ascII编码映射到对应字符后拼接字符串

C代码实现：

    #include <stdio.h>
    #include <stdlib.h>
 
    // base64 转换表, 共64个
    static const char base64_alphabet[] = {
        'A', 'B', 'C', 'D', 'E', 'F', 'G',
        'H', 'I', 'J', 'K', 'L', 'M', 'N',
        'O', 'P', 'Q', 'R', 'S', 'T',
        'U', 'V', 'W', 'X', 'Y', 'Z',
        'a', 'b', 'c', 'd', 'e', 'f', 'g',
        'h', 'i', 'j', 'k', 'l', 'm', 'n',
        'o', 'p', 'q', 'r', 's', 't',
        'u', 'v', 'w', 'x', 'y', 'z',
        '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
        '+', '/'};
 
    // 解码时使用    base64DecodeChars
    static const unsigned char base64_suffix_map[256] = {
        255, 255, 255, 255, 255, 255, 255, 255, 255, 255, 253, 255,
        255, 253, 255, 255, 255, 255, 255, 255, 255, 255, 255, 255,
        255, 255, 255, 255, 255, 255, 255, 255, 253, 255, 255, 255,
        255, 255, 255, 255, 255, 255, 255,  62, 255, 255, 255,  63,
        52,  53,  54,  55,  56,  57,  58,  59,  60,  61, 255, 255,
        255, 254, 255, 255, 255,   0,   1,   2,   3,   4,   5,   6,
        7,   8,   9,  10,  11,  12,  13,  14,  15,  16,  17,  18,
        19,  20,  21,  22,  23,  24,  25, 255, 255, 255, 255, 255,
        255,  26,  27,  28,  29,  30,  31,  32,  33,  34,  35,  36,
        37,  38,  39,  40,  41,  42,  43,  44,  45,  46,  47,  48,
        49,  50,  51, 255, 255, 255, 255, 255, 255, 255, 255, 255,
        255, 255, 255, 255, 255, 255, 255, 255, 255, 255, 255, 255,
        255, 255, 255, 255, 255, 255, 255, 255, 255, 255, 255, 255,
        255, 255, 255, 255, 255, 255, 255, 255, 255, 255, 255, 255,
        255, 255, 255, 255, 255, 255, 255, 255, 255, 255, 255, 255,
        255, 255, 255, 255, 255, 255, 255, 255, 255, 255, 255, 255,
        255, 255, 255, 255, 255, 255, 255, 255, 255, 255, 255, 255,
        255, 255, 255, 255, 255, 255, 255, 255, 255, 255, 255, 255,
        255, 255, 255, 255, 255, 255, 255, 255, 255, 255, 255, 255,
        255, 255, 255, 255, 255, 255, 255, 255, 255, 255, 255, 255,
        255, 255, 255, 255, 255, 255, 255, 255, 255, 255, 255, 255,
        255, 255, 255, 255 };
 
    static char cmove_bits(unsigned char src, unsigned lnum, unsigned rnum) {
        src <<= lnum; 
        src >>= rnum;
        return src;
    }
 
    int base64_encode(  char *indata, int inlen, char *outdata, int *outlen) {
    
        int ret = 0; // return value
        if (indata == NULL || inlen == 0) {
            return ret = -1;
        }
    
        int in_len = 0; // 源字符串长度, 如果in_len不是3的倍数, 那么需要补成3的倍数
        int pad_num = 0; // 需要补齐的字符个数, 这样只有2, 1, 0(0的话不需要拼接, )
        if (inlen % 3 != 0) {
            pad_num = 3 - inlen % 3;
        }
        in_len = inlen + pad_num; // 拼接后的长度, 实际编码需要的长度(3的倍数)
    
        int out_len = in_len * 8 / 6; // 编码后的长度
    
        char *p = outdata; // 定义指针指向传出data的首地址
    
        //编码, 长度为调整后的长度, 3字节一组
        for (int i = 0; i < in_len; i+=3) {
            int value = *indata >> 2; // 将indata第一个字符向右移动2bit(丢弃2bit)
            char c = base64_alphabet[value]; // 对应base64转换表的字符
            *p = c; // 将对应字符(编码后字符)赋值给outdata第一字节
        
            //处理最后一组(最后3字节)的数据
            if (i == inlen + pad_num - 3 && pad_num != 0) {
                if(pad_num == 1) {
                    *(p + 1) = base64_alphabet[(int)(cmove_bits(*indata, 6, 2) + cmove_bits(*(indata + 1), 0, 4))];
                    *(p + 2) = base64_alphabet[(int)cmove_bits(*(indata + 1), 4, 2)];
                    *(p + 3) = '=';             
                } else if (pad_num == 2) { // 编码后的数据要补两个 '='
                    *(p + 1) = base64_alphabet[(int)cmove_bits(*indata, 6, 2)];
                    *(p + 2) = '=';
                    *(p + 3) = '=';
                }
            } 
            else              // 处理正常的3字节的数据
            { 
                *(p + 1) = base64_alphabet[cmove_bits(*indata, 6, 2) + cmove_bits(*(indata + 1), 0, 4)];
                *(p + 2) = base64_alphabet[cmove_bits(*(indata + 1), 4, 2) + cmove_bits(*(indata + 2), 0, 6)];
                *(p + 3) = base64_alphabet[*(indata + 2) & 0x3f];
            }
        
            p += 4;
            indata += 3;
        }
    
        if(outlen != NULL) {
            *outlen = out_len;
        }
    
        return ret;
    }
 
 
    int base64_decode(const char *indata, int inlen, char *outdata, int *outlen) 
    {
    
        int ret = 0;
        if (indata == NULL || inlen <= 0 || outdata == NULL || outlen == NULL) {
            return ret = -1;
        }
        if (inlen % 4 != 0) { // 需要解码的数据不是4字节倍数
            return ret = -2;
        }
    
        int t = 0, x = 0, y = 0, i = 0;
        unsigned char c = 0;
        int g = 3;
    
        //while (indata[x] != 0) {
        while (x < inlen) {
            // 需要解码的数据对应的ASCII值对应base64_suffix_map的值
            c = base64_suffix_map[indata[x++]];
            if (c == 255) return -1;// 对应的值不在转码表中
            if (c == 253) continue;// 对应的值是换行或者回车
            if (c == 254) { c = 0; g--; }// 对应的值是'='
            t = (t<<6) | c; // 将其依次放入一个int型中占3字节
            if (++y == 4) {
                outdata[i++] = (unsigned char)((t>>16)&0xff);
                if (g > 1) outdata[i++] = (unsigned char)((t>>8)&0xff);
                if (g > 2) outdata[i++] = (unsigned char)(t&0xff);
                y = t = 0;
            }
        }   
        if (outlen != NULL) {
            *outlen = i;
        }
        return ret;
    }


参考链接

[base64编码解码原理和C语言实现](https://www.cnblogs.com/hwd00001/p/17393159.html)

[彻底弄懂base64的编码与解码原理](https://juejin.cn/post/6994612829437296647)