---
layout: post
title: 安卓逆向入门学习
subtitle: 
categories: [other]
---

第一次接触Android apk逆向，这次的工具、题目全部来自范神，
非常感谢范神亲自给我们上课，带领大家学习apk逆向知识。
由于课上时间有限，好多知识没有消化，再练习一下。

## 01. 01-工具教学

这个题目属于工具使用教学，将apk用 Android killer 或
JEB工具打开后，能够直接看到账号和密码。

```java
private static final String REAL_PWD = "123456";
private static final String REAL_USERNAME = "282921012";
```

在雷电模拟器中安装apk，运行输入账号和密码，提示 登录成功。

## 02. crackme1-字符串

这个题目也比较简单，flag隐藏在函数`k()` 中，关键行见注释

```java
public static String k() {
    StringBuilder v1 = new StringBuilder();
    String v2 = "I am the flag";
    int[] v3 = new int[]{0x7B, 103, 72, 42, 100, 50, 94, 51, 70, 0x5F, 52, 53, 56, 0x75, 57, 52, 94, 0x7D}; // 字符的 十进制和十六进制数
    int v4 = v3.length;
    int v0;
    for(v0 = 0; v0 < v4; ++v0) {
        v1.append(Character.toString(((char)v3[v0]))); // 将int数组逐位转换为ASCII字符
    }

    return v2.substring(9, 13) + v1.toString(); // 拼接v2中的'flag' 和 上面转换好的字符串
}
```

理清楚代码逻辑，我们需要把可见的字符数组还原为字符串，并在前面加"flag"。  

Python 代码如下：

```python
c = [0x7B, 103, 72, 42, 100, 50, 94, 51, 70, 0x5F, 52, 53, 56, 0x75, 57, 52, 94, 0x7D]
print('flag'+ ''.join([chr(i) for i in m]))
# flag{gH*d2^3F_458u94^}
```

得到flag为: `flag{gH*d2^3F_458u94^}`

填到app中验证一下，可以得到密码正确的提示。

## 03. 17逆向key-逆序_异或

这个题目，提交输入经过`CheckSn()`函数的加密，
再与字符串``"x60CA6F@7=4D17G2C~|`n"`` 比较结果

CheckSn函数代码如下：

```java
static StringBuilder CheckSn(String arg5) {
    StringBuilder v2 = new StringBuilder();
    int v1 = arg5.length();
    int v0 = 0;
    while(v1 > 0) {
        v2.append(arg5.charAt(v1 - 1)); // 将arg5 逆序
        ++v0;
        --v1;
    }

    arg5 = v2.toString();
    v1 = arg5.length();
    v2.delete(0, v2.length());
    for(v0 = 0; v0 < v1; ++v0) {
        v2.append(((char)(arg5.charAt(v0) ^ 5))); // 将arg 5 逐位 异或 5
    }

    return v2;
}
```

可以看出，输入字符先进行逆序操作，再逐位与 5 异或。
解码的过程为 先与5异或，再逆序。

```python
c = "x60CA6F@7=4D17G2C~|`n"
m = [chr(ord(i) ^ 5) for i in c][::-1]
print(''.join(m))
# key{F7B24A182EC3DF53}
```

得到flag为：key{F7B24A182EC3DF53}

## 04.re1-base64_逆序

这个题目跟前面的题目稍有区别，前面的题目中，与我们输入比较的目标字符串，都写在代码中了，
我们只需要还原其中的加密过程，就可以得到结果。  

本题中，目标字符串隐藏起来了，是从`getFlag()`函数中得到的。 
拿到flag字符串后，再进行逆序-再base64解密。 代码如下：

```java

public void checkPassword(String arg4) {
    if(arg4.equals(new String(Base64.decode(new StringBuffer(this.getFlag()).reverse().toString(), 0)))) {
        this.showMsgToast("Congratulations !");
    }
    else {
        this.showMsgToast("Try again.");
    }
}

private String getFlag() {
    return this.getBaseContext().getString(0x7F0B0020);
}

```

如何获取flag字符串，是我们解题的关键，这里有两个方法可以得到

### 方法一

题目中给出了`getString(0x7F0B0020)`的线索，我们在原码中搜索`0x7F0B0020`
可以找到这个地址与 toString 这个名称有关。

```java
.field public static final toString:I = 0x7F0B0020
```

再到 解析的 APK 目录中 Resources 文件夹 找到 values 目录，找到其中的 strings.xml 文件
在这个文件中搜索 `toString` ， 可以打到目标字符串
`991YiZWOz81ZhFjZfJXdwk3X1k2XzIXZIt3ZhxmZ`

### 方法二

`getFlag` 函数的功能就是返回flag字符串，我们找到`getFlag()`函数的源码位置，
在 return-object v0 处下断点，运行程序，随便提交个输入，当程序执行到这个位置时，
在局部变量中，查看变量 v0 的值，即为`991YiZWOz81ZhFjZfJXdwk3X1k2XzIXZIt3ZhxmZ`

```java
.method private getFlag()String
          .registers 3
00000000  invoke-virtual      MainActivity->getBaseContext()Context, p0
00000006  move-result-object  v0
00000008  const               v1, 0x7F0B0020
0000000E  invoke-virtual      Context->getString(I)String, v0, v1
00000014  move-result-object  v0
00000016  return-object       v0
.end method
```

有了目标字符串，我们将字符串逆序，再解base64，就可以得到flag

```python
from base64 import b64decode
c = "991YiZWOz81ZhFjZfJXdwk3X1k2XzIXZIt3ZhxmZ"
m = b64decode(c[::-1]).decode()
print(m)
# flag{Her3_i5_y0ur_f1ag_39fbc_}
```

## 05. re4-base64

这个题目不难，也是一个base64。

代码如下：

```java
protected void onCreate(Bundle arg3) {
    super.onCreate(arg3);
    this.setContentView(0x7F04001B);
    this.findViewById(0x7F0B0076).setOnClickListener(new View$OnClickListener() {
        public void onClick(View arg8) {
            if(new Base64New().Base64Encode(MainActivity.this.findViewById(0x7F0B0075).getText().toString().getBytes()).equals("5rFf7E2K6rqN7Hpiyush7E6S5fJg6rsi5NBf6NGT5rs=")) {
                Toast.makeText(MainActivity.this, "验证通过!", 1).show();
            }
            else {
                Toast.makeText(MainActivity.this, "验证失败!", 1).show();
            }
        }
    });
}
```

上面的代码中，可以找到密文 `5rFf7E2K6rqN7Hpiyush7E6S5fJg6rsi5NBf6NGT5rs=`  

直接将base64解码是解不开的，因为上面的`Base64New()` 是自定义的base64加密过程。

查看代码如下：

```java
static {
    Base64New.Base64ByteToStr = new char[]{'v', 'w', 'x', 'r', 's', 't', 'u', 'o', 'p', 'q', '3', '4', '5', '6', '7', 'A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J', 'y', 'z', '0', '1', '2', 'P', 'Q', 'R', 'S', 'T', 'K', 'L', 'M', 'N', 'O', 'Z', 'a', 'b', 'c', 'd', 'U', 'V', 'W', 'X', 'Y', 'e', 'f', 'g', 'h', 'i', 'j', 'k', 'l', 'm', 'n', '8', '9', '+', '/'};
    Base64New.StrToBase64Byte = new byte[0x80];
}

public Base64New() {
    super();
}

public String Base64Encode(byte[] arg9) {
    int v7 = 3;
    StringBuilder v3 = new StringBuilder();
    int v1;
    for(v1 = 0; v1 <= arg9.length - 1; v1 += 3) {
        byte[] v0 = new byte[4];
        byte v4 = 0;
        int v2;
        for(v2 = 0; v2 <= 2; ++v2) {
            if(v1 + v2 <= arg9.length - 1) {
                v0[v2] = ((byte)((arg9[v1 + v2] & 0xFF) >>> v2 * 2 + 2 | v4));
                v4 = ((byte)(((arg9[v1 + v2] & 0xFF) << (2 - v2) * 2 + 2 & 0xFF) >>> 2));
            }
            else {
                v0[v2] = v4;
                v4 = 0x40;
            }
        }

        v0[v7] = v4;
        for(v2 = 0; v2 <= v7; ++v2) {
            if(v0[v2] <= 0x3F) {
                v3.append(Base64New.Base64ByteToStr[v0[v2]]);
            }
            else {
                v3.append('=');
            }
        }
    }

    return v3.toString();
}

```

上面的代码中，我们可以拿到base64的码表：

`vwxrstuopq34567ABCDEFGHIJyz012PQRSTKLMNOZabcdUVWXYefghijklmn89+/=`

使用CyberChef解码：
[base64_自定义码表](https://gchq.github.io/CyberChef/#recipe=From_Base64%28'vwxrstuopq34567ABCDEFGHIJyz012PQRSTKLMNOZabcdUVWXYefghijklmn89%2B/',true%29&input=NXJGZjdFMks2cnFON0hwaXl1c2g3RTZTNWZKZzZyc2k1TkJmNk5HVDVycz0)

也可以用Python代码来解：

```python

def base64_decode(string, base64_list=""):
    oldstr = ''
    if not base64_list:  # 如果没有自定义码表，用标准码表
        base64_list = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/="
    for i in string.replace(base64_list[-1],''): # 替换掉末尾的字符
        # 查表将字符串转为6位一组的二进制串后拼接
        oldstr += bin(base64_list.index(i)).replace('0b','').rjust(6,'0') 
    # 将二进制8位一组转换整数，然后转字符，不够8位的舍弃
    newstr = [chr(int(oldstr[j:j + 8],2)) for j in range(0, len(oldstr), 8) if j <= len(oldstr) - 8]
    return  ''.join(newstr)

c = "5rFf7E2K6rqN7Hpiyush7E6S5fJg6rsi5NBf6NGT5rs="
base64_list = "vwxrstuopq34567ABCDEFGHIJyz012PQRSTKLMNOZabcdUVWXYefghijklmn89+/="
m = base64_decode(c,base64_list)
print(m)
# 05397c42f9b6da593a3644162d36eb01
```

flag为： 05397c42f9b6da593a3644162d36eb01
输入模拟器，提示 验证通过。

## 06. app1-异或

这个题目跟上一题差不多，也是需要找到两个隐藏的参数。
关键代码如下：

```java
public void onClick(View arg10) {
    try {
        String v1 = MainActivity.this.text.getText().toString();
        PackageInfo v2 = MainActivity.this.getPackageManager().getPackageInfo("com.example.yaphetshan.tencentgreat", 0x4000);
        String v3 = v2.versionName;
        int v4 = v2.versionCode;
        int v0 = 0;
        while(v0 < v1.length()) {
            if(v0 >= v3.length()) {
                break;
            }

            if(v1.charAt(v0) != (v3.charAt(v0) ^ v4)) { // versionName 异或 versionCode 后 判断与输入字符串是否相等 
                Toast.makeText(MainActivity.this, "再接再厉，加油~", 1).show();
                return;
            }
            else {
                ++v0;
                continue;
            }
        }

        Toast.makeText(MainActivity.this, "恭喜开启闯关之门！", 1).show();
        return;
    }
    catch(PackageManager$NameNotFoundException v5) {
    }
}
```

代码的主要逻辑为，versionName字符串异或整数versionCode后，
逐位与输入字符串比较。
问题的关键是找出versionName 和 versionCode，
这两个参数都写在BuildConfig中，打开就能看到。

```java

public final class BuildConfig {
    public static final String APPLICATION_ID = "com.example.yaphetshan.tencentgreat";
    public static final String BUILD_TYPE = "debug";
    public static final boolean DEBUG = false;
    public static final String FLAVOR = "";
    public static final int VERSION_CODE = 15;
    public static final String VERSION_NAME = "X<cP[?PHNB<P?aj";
}

```

也可以尝试通过在`onClick()`函数中下断点来找出versionName和versionCode，有了这两个参数，
执行异或操作，就可以拿到flag  
代码如下：

```python
versionName = "X<cP[?PHNB<P?aj"
versionCode = 15
m = [chr(ord(i) ^ versionCode) for i in versionName]
print("".join(m))
# W3l_T0_GAM3_0ne
```

flag为： W3l_T0_GAM3_0ne

## 07. 08-base64_值减5_首尾互调

这道题的加密过程稍微有点绕，主要加密函数如下：

```java
public static String Work(String arg8) {
    String v4 = Base64.encodeToString(arg8.getBytes(), 2); // 将输入base64编码
    int v2 = v4.length();
    char[] v0 = new char[v2];
    int v1;
    for(v1 = 0; v1 < v2; ++v1) {
        v0[v1] = v4.charAt(v1);
    }

    for(v1 = 0; v1 < v2 / 2; ++v1) {      // 取前半部分
        char v5 = ((char)(v0[v1] - 5));   // 将字符值减去5
        v0[v1] = v0[v2 - v1 - 1];         // 将首尾的值互换 123456 换为 654321
        v0[v2 - v1 - 1] = v5;   
    }
    return new String(v0);
}
```

这个函数我刚开始没看懂，后来又一句一句的比划了半天，才弄明白。
过程如下：

1. 先将字符串进行base64加密
2. 加密后的字符串，取前一半，ascii值减去5
3. 将前半段字符与后半段字符首尾互调

把上面的加密逻辑理清后，从主函数中找到加密后的字符串`"==wMGBjNFJTR,LOL=UOMsD@H"`
我们可以倒序过程解密

```python
from base64 import b64decode
c = list("==wMGBjNFJTR,LOL=UOMsD@H")
for i in range(len(c) // 2):
    c[i], c[-i-1] = c[-i-1], c[i]
    c[i] = chr(ord(c[i]) + 5)
m = b64decode(''.join(c)).decode()
print(m)
# 0B1E6AA45E2E60F3
```

最终结果为： 0B1E6AA45E2E60F3
输入模拟器验证，提示 激活码正确

## 08. 09_暴力破解

这个题目有一定的难度，考察一种完全不同的思路。
主要加密过程如下：

```java

public class encode {
    private static byte[] b;

    static {
        encode.b = new byte[]{23, 22, 26, 26, 25, 25, 25, 26, 27, 28, 30, 30, 29, 30, 0x20, 0x20}; // 给定数组
    }

    public encode() {
        super();
    }

    public static boolean check(String arg7) {
        int v6 = 16;
        byte[] v1 = arg7.getBytes();
        byte[] v3 = new byte[v6];
        int v0;
        for(v0 = 0; v0 < v6; ++v0) {
            v3[v0] = ((byte)((v1[v0] + encode.b[v0]) % 61));  // 输入字符串与标准字符串进行 相加 取余 得到新串 v3
        }

        for(v0 = 0; v0 < v6; ++v0) {
            v3[v0] = ((byte)(v3[v0] * 2 - v0));  // 将v3 字符值 * 2 再减去索引值
        }

        return new String(v3).equals(arg7);  // 判断输入字符串与加密变化过的字符是否相等
    }
}
```

代码过程：

1. 将输入字符串逐位与给定字符串相加，再对61取余，得到新的字符串v3
2. 将v3字符串值 乘以 2 再减去索引值
3. 将最终结果与输入比较，如果相等，就正确。

前面的题目中，加密过程一般都是 +- 数字、异或、base64等，这些都是可逆的。
本题中用到的 取余过程，是一个不可逆的过程。
而且是对输入进行运算，运算结果再和输出进行比较，
如果输入不正确，运算结果也不正确，下断点也行不通。

范神提示我们，既然输入 运算 再与输入比较 这个过程我们是知道的，
肯定存在一个字符串能满足这个要求，那就爆破呀。 一共就16位，也不长。

python爆破代码如下：

```python
from string import printable

b = [23, 22, 26, 26, 25, 25, 25, 26, 27, 28, 30, 30, 29, 30, 0x20, 0x20]
flag = ''
for i in range(16):
    for j in printable:
        v3 = ((ord(j) + b[i]) % 61) * 2 - i
        if ord(j) == v3:
            flag += j
            break
print(flag)

# LOHILMNMLKHILKHI
```

爆破结果为: LOHILMNMLKHILKHI
输入模拟器验证，提示 correct

## 09. 06-逆序_md5_base64 

在JEB中打开apk，查看主函数，关键行如下：

```java
MainActivity.this.str.reverse();  // 逆序
Log.i("ClownQiang", "str--->" + new String(MainActivity.this.str));
String v1 = MainActivity.encode(new String(MainActivity.this.str)); // encode加密后赋值给v1 查看encode函数，加密过程为md5
Log.i("ClownQiang", "base64--->" + MainActivity.this.str);
String v0 = MainActivity.getBASE64(v1).trim(); // base64后 赋值给v0
Log.i("ClownQiang", "md5--->" + v1);
if(v0.equalsIgnoreCase("NzU2ZDJmYzg0ZDA3YTM1NmM4ZjY4ZjcxZmU3NmUxODk=")) { // 判断v0的值是否等于字符串
    MainActivity.this.dialog2.showDialog();
}
```

看清楚上面的逻辑后，我们的解密过程如下

1. 将`NzU2ZDJmYzg0ZDA3YTM1NmM4ZjY4ZjcxZmU3NmUxODk=` base64解码，得到md5序列
2. 将`756d2fc84d07a356c8f68f71fe76e189` 进行解密。 
3. md5解密结果为`}321nimda{galflj` 再逆序得到最终的flag: `jlflag{admin123}`

找了好久，终于找到一个免费的[在线MD5解密](https://www.somd5.com/)
将拿到的结果软件中提交一下，可以看到`congratulations, you success!!!` 的提示

## 10. 11-修改程序if判断

这个题难度升级了，代码读起来很有难度，反正我是没有看懂。。。  

代码如下：

```java
package com.example.findit;

import android.os.Bundle;
import android.support.v7.app.ActionBarActivity;
import android.view.MenuItem;
import android.view.View$OnClickListener;
import android.view.View;

public class MainActivity extends ActionBarActivity {
    public MainActivity() {
        super();
    }

    protected void onCreate(Bundle arg8) {
        super.onCreate(arg8);
        this.setContentView(0x7F030018);
        this.findViewById(0x7F05003D).setOnClickListener(new View$OnClickListener(new char[]{'T', 'h', 'i', 's', 'I', 's', 'T', 'h', 'e', 'F', 'l', 'a', 'g', 'H', 'o', 'm', 'e'}, this.findViewById(0x7F05003E), new char[]{'p', 'v', 'k', 'q', '{', 'm', '1', '6', '4', '6', '7', '5', '2', '6', '2', '0', '3', '3', 'l', '4', 'm', '4', '9', 'l', 'n', 'p', '7', 'p', '9', 'm', 'n', 'k', '2', '8', 'k', '7', '5', '}'}, this.findViewById(0x7F05003F)) {
            public void onClick(View arg13) {
                int v11 = 17;
                int v10 = 0x7A;
                int v9 = 90;
                int v8 = 65;
                int v7 = 97;
                char[] v3 = new char[v11];
                char[] v4 = new char[38];
                int v0;
                for(v0 = 0; v0 < v11; ++v0) {
                    if(this.val$a[v0] >= 73 || this.val$a[v0] < v8) {
                        if(this.val$a[v0] < 105 && this.val$a[v0] >= v7) {
                        label_39:
                            v3[v0] = ((char)(this.val$a[v0] + 18));
                            goto label_44;
                        }

                        if(this.val$a[v0] >= v8 && this.val$a[v0] <= v9 || this.val$a[v0] >= v7 && this.val$a[v0] <= v10) {
                            v3[v0] = ((char)(this.val$a[v0] - 8));
                            goto label_44;
                        }

                        v3[v0] = this.val$a[v0];
                    }
                    else {
                        goto label_39;
                    }

                label_44:
                }

                if(String.valueOf(v3).equals(this.val$edit.getText().toString())) {
                    v0 = 0;
                    goto label_18;
                }
                else {
                    this.val$text.setText("答案错了肿么办。。。不给你又不好意思。。。哎呀好纠结啊~~~");
                    return;
                label_18:
                    while(v0 < 38) {
                        if(this.val$b[v0] < v8 || this.val$b[v0] > v9) {
                            if(this.val$b[v0] >= v7 && this.val$b[v0] <= v10) {
                            label_80:
                                v4[v0] = ((char)(this.val$b[v0] + 16));
                                if((v4[v0] <= v9 || v4[v0] >= v7) && v4[v0] < v10) {
                                    goto label_95;
                                }

                                v4[v0] = ((char)(v4[v0] - 26));
                                goto label_95;
                            }

                            v4[v0] = this.val$b[v0];
                        }
                        else {
                            goto label_80;
                        }

                    label_95:
                        ++v0;
                    }

                    this.val$text.setText(String.valueOf(v4));
                }
            }
        });
    }

    public boolean onOptionsItemSelected(MenuItem arg3) {
        boolean v1 = arg3.getItemId() == 0x7F050040 ? true : super.onOptionsItemSelected(arg3);
        return v1;
    }
}


```

但是范神教导我们，没有看懂也没有关系。不需要看懂所有的代码，
只需要看懂框架，知道大致的流程。
这道题中我们需要看懂的地方是：  

1. 题目给定的字符串经过一系列运算，得到v3字符串
2. 将用户输入的字符串与v3比较，如果不一致，就弹出`答案错了肿么办。...`
3. 如果一致，则算出一个flag，显示出来。

那解题的关键就是步骤2中的if判断。我们看不懂v3和flag怎么算出来的不要紧，
只需要将**if**的判断条件**取反**就行。  
这样题目的逻辑就变成，如果输入错误，就弹出flag。

具体的操作方法有两种。

第一种，使用JEB在判断语句下断点，运行到断点处，修改局部变量中v5的值。 0改为1，继续运行。

```java
00000044  invoke-virtual      String->equals(Object)Z, v1, v5
0000004A  move-result         v5
0000004C  if-eqz              v5, :190  //下断点处
```

第二种，使用andriod Killer 工具，将if-eqz修改为if-ne，重新编译。

得到flag为：flag{c164675262033b4c49bdf7f9cda28a75}

## 11. 15-AES解密

这个题目换了一种加密方法，AES加密,
关键的加密代码如下：

```java
public class b {
    private String ivParameter;
    private String sKey;

    public b() {
        super();
        this.sKey = "EaRncVfLgIPMaygA";
        this.ivParameter = "1234567890123456";
    }

    public String encrypt(String arg7) throws Exception {
        Cipher v0 = Cipher.getInstance("AES/CBC/PKCS5Padding");
        v0.init(1, new SecretKeySpec(this.sKey.getBytes(), "AES"), new IvParameterSpec(this.ivParameter.getBytes()));
        return Base64.encodeToString(v0.doFinal(arg7.getBytes("utf-8")), 0);
    }
}

```

从上面的代码中，可以直接看到的信息有 加密方法AES，密钥，加密向量，加密向量

加密后的密文在主函数的代码中,可以直接拿到
`"kt1zBVG4Om1rVxe2ISqt1WFZ+tWvgNV3QygfrSlFRjI="`

好了，只需要解出密文就行了。需要注意的一点是加密后的结果又进行base64加密。

用cyberchef 工具解密比较快，解密的链接如下：

[15.apk链接](https://gchq.github.io/CyberChef/#recipe=From_Base64%28'A-Za-z0-9%2B/%3D',true%29AES_Decrypt%28%7B'option':'UTF8','string':'EaRncVfLgIPMaygA'%7D,%7B'option':'UTF8','string':'1234567890123456'%7D,'CBC','Raw','Raw',%7B'option':'Hex','string':''%7D%29&input=a3QxekJWRzRPbTFyVnhlMklTcXQxV0ZaK3RXdmdOVjNReWdmclNsRlJqST0)

参数设置为 {key:"utf-8", iv:"utf-8", input:"Raw", Mode:"CBC"}，

解密结果为：18964C5211863BB9

也可以写python来解，代码如下:

```python
import base64
from Crypto.Cipher import AES

def AES_Decrypt(key, data):
    iv = '1234567890123456'
    data = base64.b64decode(data.encode())
    cipher = AES.new(key.encode(),AES.MODE_CBC,iv.encode())
    text_decrypted = cipher.decrypt(data)
    return text_decrypted.decode()

key = "EaRncVfLgIPMaygA"
data = "kt1zBVG4Om1rVxe2ISqt1WFZ+tWvgNV3QygfrSlFRjI="
text_decrypted = AES_Decrypt(key,data)
print(text_decrypted)
# 18964C5211863BB9
```

## 12. AFERE-DES_base64

这个题目难度又提升了一大截。  本题APK 扔进JEB打不开。
APK格式实际是ZIP，可以用ZIP解压，既然是ZIP格式，就少不了zip伪加密
本题的apk是伪加密的，需要先用zip伪加密去除工具，
范神给介绍了两个工具：ApkEntTool伪加密 和 ZipCenOp伪加密
也可以用010Editer 来手工解伪加密
解开后，能看到代码

代码如下：

```java
public class MainActivity extends Activity {
    private static final char[] a;

    static {
        MainActivity.a = "S4wp902KOV7QRogXdIUCMW1/ktz8sa5c3xePGfENuDTvBFqAmrbnLlHZYyhJij6+".toCharArray();
    }

    public MainActivity() {
        super();
    }

    public String b(byte[] arg19) throws InvalidKeyException, NoSuchAlgorithmException, NoSuchPaddingException, InvalidKeySpecException, IllegalBlockSizeException, BadPaddingException {
        int v1;
        int v10;
        int v9;
        SecureRandom v14 = new SecureRandom();
        DESKeySpec v12 = new DESKeySpec("Mem3d4Da".getBytes());
        Cipher v6 = Cipher.getInstance("DES");
        v6.init(1, SecretKeyFactory.getInstance("DES").generateSecret(((KeySpec)v12)), v14);
        byte[] v7 = v6.doFinal(arg19);
        int v13 = v7.length;
        char[] v3 = new char[(v13 + 2) / 3 * 4];
        int v5 = 0;
        int v2 = 0;
        while(v5 < v13) {
            int v4 = v5 + 1;
            int v8 = v7[v5];
            if(v4 < v13) {
                v5 = v4 + 1;
                v9 = v7[v4];
            }
            else {
                v9 = 0;
                v5 = v4;
            }

            if(v5 < v13) {
                v4 = v5 + 1;
                v10 = v7[v5];
            }
            else {
                v10 = 0;
                v4 = v5;
            }

            v1 = v2 + 1;
            v3[v2] = MainActivity.a[v8 >> 2 & 0x3F];
            v2 = v1 + 1;
            v3[v1] = MainActivity.a[(v8 << 4 | (v9 & 0xFF) >> 4) & 0x3F];
            v1 = v2 + 1;
            v3[v2] = MainActivity.a[(v9 << 2 | (v10 & 0xFF) >> 6) & 0x3F];
            v2 = v1 + 1;
            v3[v1] = MainActivity.a[v10 & 0x3F];
            v5 = v4;
        }

        switch(v13 % 3) {
            case 1: {
                goto label_80;
            }
            case 2: {
                goto label_87;
            }
        }

        goto label_30;
    label_87:
        v1 = v2;
        goto label_83;
    label_80:
        v1 = v2 - 1;
        v3[v1] = '*';
    label_83:
        v3[v1 - 1] = '*';
    label_30:
        return new String(v3);
    }

    public void confirm(View arg6) throws InvalidKeyException, NoSuchAlgorithmException, NoSuchPaddingException, InvalidKeySpecException, IllegalBlockSizeException, BadPaddingException {
        View v3 = this.findViewById(0x7F05003D);
        if(this.b(this.findViewById(0x7F05003C).getText().toString().getBytes()).equals("OKBvTrSKXPK3cObqoS21IW7Dg0eZ2RTYm3UrdPaVTdY*")) {
            ((TextView)v3).setText("You find the Key");
        }
        else {
            ((TextView)v3).setText("Wrong");
        }
    }

    protected void onCreate(Bundle arg2) {
        super.onCreate(arg2);
        this.setContentView(0x7F030018);
    }
}
```

第二个难点为，中间的加密过程是base64，只不过码表是自定义的。
由于只是大概了解bas64，所以我没有看出来，
平时只会用工具解base64编码，惭愧惭愧，

从上面的代码中看出，先DES加密，然后base64，可以找到相关信息：

1. base64码表：`"S4wp902KOV7QRogXdIUCMW1/ktz8sa5c3xePGfENuDTvBFqAmrbnLlHZYyhJij6+*"`
2. DES加密密钥：`Mem3d4Da`
3. 密文：`OKBvTrSKXPK3cObqoS21IW7Dg0eZ2RTYm3UrdPaVTdY*`

然后开始解密

cyberchef解密链接如下
[base64+DES](https://gchq.github.io/CyberChef/#recipe=From_Base64%28'S4wp902KOV7QRogXdIUCMW1/ktz8sa5c3xePGfENuDTvBFqAmrbnLlHZYyhJij6%2B*',true%29DES_Decrypt%28%7B'option':'UTF8','string':'Mem3d4Da'%7D,%7B'option':'Hex','string':''%7D,'ECB','Raw','Raw'%29&input=T0tCdlRyU0tYUEszY09icW9TMjFJVzdEZzBlWjJSVFltM1VyZFBhVlRkWSo)

结果为: ISG{f4kE3ncRyP71On!50ld}

用Python 解题代码如下：

```python
from pyDes import *
from binascii import b2a_hex, a2b_hex

def base64_decrypt(string, base64_list=""):
    oldstr = ''
    if not base64_list:  # 如果没有自定义码表，用标准码表
        base64_list = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/="
    for i in string.replace(base64_list[-1],''): # 替换掉末尾的字符
        # 查表将字符串转为6位一组的二进制串后拼接
        oldstr += bin(base64_list.index(i)).replace('0b','').rjust(6,'0') 
    # 将二进制8位一组转换整数，然后转16进制串，不够8位的舍弃
    newstr = [hex(int(oldstr[j:j + 8],2))[2:].rjust(2,'0') for j in range(0, len(oldstr), 8) if j <= len(oldstr) - 8]
    return  ''.join(newstr)

def des_decrypt(data, key):
    k = des(Des_Key, ECB, padmode=PAD_PKCS5)
    return k.decrypt(a2b_hex(data)) # 将十六进制串转换为bytes再传入解密

base64_list = 'S4wp902KOV7QRogXdIUCMW1/ktz8sa5c3xePGfENuDTvBFqAmrbnLlHZYyhJij6+*'
c = "OKBvTrSKXPK3cObqoS21IW7Dg0eZ2RTYm3UrdPaVTdY*"
key = "Mem3d4Da"

m = base64_decrypt(c, base64_list) 
print(m)
# 207b2bab10073e31e07c8cae3401964552a93858b718cab8c204b1423749a90e
flag = des_decrypt(m, key).decode()
print(flag)
# ISG{f4kE3ncRyP71On!50ld}
```

## 13. 13-调试编译

这道题主要考察程序的断点调试或反编译，代码如下

```java
public void onClick(View arg5) {
    System.loadLibrary("Pwd");
    String v0 = MainActivity.this.password.getText().toString().trim();
    if("".equals(v0)) {
        Toast.makeText(MainActivity.this, "输入不能为空", 0).show();
    }
    else if(MainActivity.this.PwdFromC().equals(v0)) {
        Toast.makeText(MainActivity.this, "恭喜", 0).show();
    }
    else {
        Toast.makeText(MainActivity.this, "flag错误", 0).show();
    }
}
```

从代码中可以看出，程序的逻辑为，输入与`PwdFromC()`函数的结果相比较。
这个函数是个 `public native String PwdFromC() {}` 函数，
具体的位置在`libPwd.so` (Libraries/armeabi/libPwd.so) 这个文件中,
打开看了看，啥也看不懂。

还是用的断点调试大法，在`String->equals(Object)Z, v1, v0` 处下断，
查看 v1 的值。

```java
00000058  iget-object         v1, p0, MainActivity$MyonclickListener->this$0:MainActivity
0000005C  invoke-virtual      MainActivity->PwdFromC()String, v1
00000062  move-result-object  v1
00000064  invoke-virtual      String->equals(Object)Z, v1, v0
0000006A  move-result         v1
0000006C  if-eqz              v1, :88
```

结果为： "KnAvE"

## 14. 10-值减1_md5

这个题难度比较高，主要的难点跟上一题一样，引入了库文件`libphcm.so`，
主要的函数getFlag()和加密过程encrypt()都在库文件中。

题目的主要代码如下：

```java
public class MainActivity extends AppCompatActivity {
    EditText etFlag;

    static {
        System.loadLibrary("phcm");
    }

    public native String encrypt(String arg1) {
    }

    public native String getFlag() {
    }

    public void onGoClick(View arg5) {
        if(this.getSecret(this.getFlag()).equals(this.getSecret(this.encrypt(this.etFlag.getText().toString())))) {
            Toast.makeText(((Context)this), "Success", 1).show();
        }
        else {
            Toast.makeText(((Context)this), "Failed", 1).show();
        }
    }
}
```

题目中还给我们了getSecret()函数，但是也没看懂。  

上面代码中最关键的一句是onGoClick()函数的if判断条件：

`if(this.getSecret(this.getFlag()).equals(this.getSecret(this.encrypt(this.etFlag.getText().toString()))))`

在这一句中，equals()函数左右两边都经过了getSecret()函数的加密，相当于都没做
所以我们只需要确认输入经过encrypt()函数后等于getFlag()就行了。
我们是目标有两个：

1. 获得getFlag()函数的输出
2. 获得ecrypyt()函数的变化过程

明确目标后，我们用下断点调试的方法来查找flag。
程序对应源码如下：

```java
.method public onGoClick(View)V
          .registers 6
00000000  const/4             v3, 1
00000002  iget-object         v1, p0, MainActivity->etFlag:EditText
00000006  invoke-virtual      EditText->getText()Editable, v1
0000000C  move-result-object  v1
0000000E  invoke-virtual      Object->toString()String, v1
00000014  move-result-object  v0
00000016  invoke-virtual      MainActivity->getFlag()String, p0
0000001C  move-result-object  v1
0000001E  invoke-virtual      MainActivity->getSecret(String)String, p0, v1
00000024  move-result-object  v1
00000026  invoke-virtual      MainActivity->encrypt(String)String, p0, v0
0000002C  move-result-object  v2
0000002E  invoke-virtual      MainActivity->getSecret(String)String, p0, v2
00000034  move-result-object  v2
00000036  invoke-virtual      String->equals(Object)Z, v1, v2
0000003C  move-result         v1
0000003E  if-eqz              v1, :56
```

不论我们的输入是什么，程序都要先算出flag,
我们在`0000001E`行下断点，查看局部变量v1的值，可以找到flag
``ek`fz@q2^x/t^fn0mF^6/^rb`qanqntfg^E`hq|``
经过getFlag()函数变化后，输出为：f2a3628f6ed2b1d516a269007ba04dd6，
怀疑是md5加密。

接下来的重点是encrypt()函数。我们在`libphcm.so`文件中可以找到源码如下：

```java
unsigned int Java_com_ph0en1x_android_1crackme_MainActivity_encrypt(char* $R0, char* $R1, unsigned int $R2) {
    unsigned int $R3 = *(*((int*)(&$R0[0])) + 676);
    char* $R6 = $R0;
    $R0 = {$R3}($R0, $R2, 0, $R3, $R4);
    $R4 = $R0;
    char* $R5 = $R0;
    $R0 = _ptr_strlen($R4);

    while(((unsigned char)(((unsigned int)(((int)$R5) - ((int)$R4))) < ((int)$R0)))) {
        $R5[0] = (unsigned char)(((unsigned int)($R5[0])) - 1); //  字符值减一
        ++$R5;
        $R0 = _ptr_strlen($R4);
    }

    $R3 = *(*((int*)(&$R6[0])) + 668);
    jump {$R3};
}
```

从上面的代码中，可以看出（我没有看出来，是听了讲解才知道的），
`encrypt()`函数将输入的字符串值减1。

其实上面的过程也可以通过断点调试来印证，
我们在`0000002E`处下断点，v0的值为我们的输入，
v2的值为经过`encrypt()`函数加密处理，我们输入"12345bcd"
可以看出v0处字符串为"12345bcd"，v2处为"01234abc"，
从这点来说，我们即使看不懂`libphcm.so`中的源码，
仅从输入输出的对应关系，也可以大胆的猜测`encrypt()`函数加密过程。

解密的过程就相对简单。将找到的flag字符串逐位加1即可。

```python
c = "ek`fz@q2^x/t^fn0mF^6/^rb`qanqntfg^E`hq|"
m = [chr(ord(i) + 1) for i in c]
print("".join(m))
# flag{Ar3_y0u_go1nG_70_scarborough_Fair}
```

得到flag为：flag{Ar3_y0u_go1nG_70_scarborough_Fair}  
输入模拟器中验证一下，Success

## 15. 16-AES

这道题是压轴的题目，难度对我来说很高，主要的难点是程序逻辑混乱，
函数名、变量名瞎起，代码难以阅读。只要读懂程序代码，弄明白逻辑后，
就可以解题，所以考的就是跟踪程序执行的能力。

下面的重点就放在程序的分析上，先看程序入口
由于有许多同名的类和方法名，在注释上打一个标签。

函数主类 MainActivity

```java
public class MainActivity extends q {
    private String v;

    public MainActivity() {
        super();
    }

    static String a(MainActivity arg1) { // tag_6: 传递Key的值。
        return arg1.v;
    }

    static boolean a(MainActivity arg1, String arg2, String arg3) { // tag_5:  第2个参数为tag_6, 第3个参数为输入字符串，调用tag_7
        return arg1.a(arg2, arg3);
    }

    private boolean a(String arg4, String arg5) {  // tag_7： 创建c类实例，调用c类中a()函数，传入key 和 输入字符串。 equals 中的值为我们要比较的值
        return new c().a(arg4, arg5).equals(new String(new byte[]{21, -93, -68, -94, 86, 0x75, -19, -68, -92, 33, 50, 0x76, 16, 13, 1, -15, -13, 3, 4, 103, -18, 81, 30, 68, 54, -93, 44, -23, 93, 98, 5, 59}));
        // tag_17: 将tag_16返回字符串与 给定值相比较。
    }

    protected void onCreate(Bundle arg3) {
        super.onCreate(arg3);
        this.setContentView(0x7F04001A);
        ApplicationInfo v0 = this.getApplicationInfo();
        v0.flags &= 2;
        this.p();  // tag_0: 跳转p()
        this.findViewById(0x7F0B0055).setOnClickListener(new d(this)); // tag_2：跳转类D，将v传入。
    }

    private void p() {  // tag_1: 打开url.png，读取16个字节，存入v中
        try { 
            InputStream v0_1 = this.getResources().getAssets().open("url.png");
            int v1 = v0_1.available();
            byte[] v2 = new byte[v1];
            v0_1.read(v2, 0, v1);
            byte[] v0_2 = new byte[16];
            System.arraycopy(v2, 0x90, v0_2, 0, 16);
            this.v = new String(v0_2, "utf-8");
        }
        catch(Exception v0) {
            v0.printStackTrace();
        }
    }
}
```

类D 实现一个监听器，通过MainActivity类中的方法a(tag_3) 来返回结果。

```java
class d implements View$OnClickListener {
    d(MainActivity arg1) {
        this.a = arg1;  // tag_3: 将传入字符串赋值给a
        super();
    }

    public void onClick(View arg5) {
        if(MainActivity.a(this.a, MainActivity.a(this.a), this.a.findViewById(0x7F0B0056).getText().toString())) {  // tag_4: 调用MainActivity.a函数，传入三个参数，返回bool值
            View v0 = this.a.findViewById(0x7F0B0054);
            Toast.makeText(this.a.getApplicationContext(), "Congratulations!", 1).show();
            ((TextView)v0).setText(0x7F060022);
        }
        else {
            Toast.makeText(this.a.getApplicationContext(), "Oh no.", 1).show();
        }
    }
}
```

类A，实现一个AES加密实例。可以选择传不传key

```java
public class a {
    private SecretKeySpec a;
    private Cipher b;

    public a() {  
        super(); 
    }

    protected void a(byte[] arg4) { // tag_13: 传入key，返回一个AES加密实例
        if(arg4 != null) {
            goto label_15;
        }

        try {
            this.a = new SecretKeySpec(MessageDigest.getInstance("MD5").digest("".getBytes("utf-8")), "AES");
            this.b = Cipher.getInstance("AES/ECB/PKCS5Padding");
            return;
        label_15:
            this.a = new SecretKeySpec(arg4, "AES");  // 使用key生成加密的密钥
            this.b = Cipher.getInstance("AES/ECB/PKCS5Padding"); // 生成一个AES加密实例
        }
        catch(UnsupportedEncodingException v0) {
            v0.printStackTrace();
        }
        catch(NoSuchAlgorithmException v0_1) {
            v0_1.printStackTrace();
        }
        catch(NoSuchPaddingException v0_2) {
            v0_2.printStackTrace();
        }
    }

    protected byte[] b(byte[] arg4) { // tag_15: 使用tag_13创建的AES加密实例，加密输入的字符串
        this.b.init(1, this.a);
        return this.b.doFinal(arg4);
    }
}
```

类C，实现一个相邻字符交换的变化。

```java
public class c {
    public c() {
        super();
    }

    public String a(String arg5, String arg6) {  // tag_8： 第一个参数为key，第二个参数为输入字符串
        String v0 = this.a(arg5);  // tag_9: 调用私有函数a，将key传入，对key进行加密
        String v1 = "";
        a v2 = new a();    // tag_11: 创建 A 类的实例，V2
        v2.a(v0.getBytes());   // tag_12: 调用A类中a函数，并传入tag_10处理过的字符串key
        try {
            v0 = new String(v2.b(arg6.getBytes()), "utf-8"); // tag_14: 调用A类的b函数，处理输入字符串
        }
        catch(Exception v0_1) {
            v0_1.printStackTrace();
            v0 = v1;
        }

        return v0; // tag_16: 返回输入字符串AES加密后的结果
    }

    private String a(String arg4) { // tag_10: 将字符串以两位为单位，单位内左右互换。
        String v0_2;
        try {
            arg4.getBytes("utf-8");
            StringBuilder v1 = new StringBuilder();
            int v0_1;
            for(v0_1 = 0; v0_1 < arg4.length(); v0_1 += 2) {  // 索引每次加2，先添加v0+1 再添加v0，实现字符换位
                v1.append(arg4.charAt(v0_1 + 1)); 
                v1.append(arg4.charAt(v0_1));
            }

            v0_2 = v1.toString();
        }
        catch(UnsupportedEncodingException v0) {
            v0.printStackTrace();
            v0_2 = null;
        }

        return v0_2;
    }
}
```

我们跟踪程序的进展如下：

1. tag_0: 跳转到p()，返回字符串，做key用
2. tag_1: p()函数，从图片url.png中读取16个字节，转换为utf-8字符串
3. tag_2: 跳转类D，将读取的字符串key传入
4. tag_3: 将传入字符串赋值给this.a
5. tag_4: 关键的一步，调用MainActivity.a函数，根据返回bool值结束程序。传入前两个参数都是key，第三个参数为输入字符串
6. tag_5: 将a函数换为两个参数，第一个是key，第二个是输入字符串
7. tag_6: 传递key的值。
8. tag_7: 关键步骤，创建C类实例，调用C类public_a函数，传入key和输入字符串。 同时equals给定数组为我们最终要比较的值
9. tag_8: C类public_a函数，传入key和输入字符串
10. tag_9: 调用C类private_a函数，传入key，对key进行加密
11. tag_10: C类private_a函数 将字符串以两位为单位，每两位前后互换。
12. tag_11: 创建A类的实例，V2
13. tag_12: 调用A类中a函数，并传入tag_10处理过的字符串key
14. tag_13: A类a函数, 传入key，返回一个AES加密实例
15. tag_14: 调用A类的b函数，传入输入字符串
16. tag_15: 使用tag_13创建的AES加密实例，加密输入的字符串
17. tag_16: 返回输入字符串AES加密后的结果
18. tag_17: AES加密结果与equals()中给定值相比较。

其中主要的程序逻辑就是，将输入字符串进行AES加密，与给定字符比较。
我们的目标是找到AES加密的key 和 给定的字符串。  

在 tag_4附近下断点(`00000038`)，查看v2的值，可以看到 key："this_is_the_key."

```java
.method public onClick(View)V
          .registers 6
00000000  const/4             v3, 1
00000002  iget-object         v0, p0, d->a:MainActivity
00000006  const               v1, 0x7F0B0056
0000000C  invoke-virtual      MainActivity->findViewById(I)View, v0, v1
00000012  move-result-object  v0
00000014  check-cast          v0, EditText
00000018  invoke-virtual      EditText->getText()Editable, v0
0000001E  move-result-object  v0
00000020  invoke-virtual      Object->toString()String, v0
00000026  move-result-object  v0
00000028  iget-object         v1, p0, d->a:MainActivity
0000002C  iget-object         v2, p0, d->a:MainActivity
00000030  invoke-static       MainActivity->a(MainActivity)String, v2
00000036  move-result-object  v2
00000038  invoke-static       MainActivity->a(MainActivity, String, String)Z, v1, v2, v0
0000003E  move-result         v0
00000040  if-eqz              v0, :86
```

但这个key还需要变化一下。

在tag_9附近下断点（`00000008`），查看v0值，可以看到变化后的key为：
`"htsii__sht_eek.y"`

```java
.method public a(String, String)String
          .registers 7
00000000  invoke-direct       c->a(String)String, p0, p1
00000006  move-result-object  v0
00000008  const-string        v1, ""
0000000C  new-instance        v2, a
00000010  invoke-direct       a-><init>()V, v2
00000016  invoke-virtual      String->getBytes()[B, v0
0000001C  move-result-object  v0
0000001E  invoke-virtual      a->a([B)V, v2, v0
```

有了key的值，接下来就需要找加密序列了。
给定的数组中有正数和负数，这里需要将有符号整数转无符号整数
具体的知识点自行搜索学习，这里只说我们需要的结论，
有符号数与无符号数之间的转换，都要看要转换的数的最高位是否为1，
如果不为1，则转换结果就是要转换的数的本身；
如果为1，则转换结果就是转换的数（看作是负数）的补码
具体到这个题目：
无符号数 = ( 有符号数 + 2 ^ N )mod 2 ^ N

下面是我们的代码：

```python

from Crypto.Cipher import AES
from binascii import b2a_hex, a2b_hex
a = [21, -93, -68, -94, 86, 117, -19, -68, -92, 33, 50, 118, 16, 13, 1, -
     15, -13, 3, 4, 103, -18, 81, 30, 68, 54, -93, 44, -23, 93, 98, 5, 59]
t = ''.join([hex((i + 256) % 256)[2:].rjust(2, '0') for i in a])
print(t)
# 15a3bca25675edbca4213276100d01f1f3030467ee511e4436a32ce95d62053b
c = a2b_hex(hex(int('0x' + t,16))[2:]) # 转换为字节码
def AES_Decrypt(key, data):
    cipher = AES.new(key.encode(),AES.MODE_ECB)
    text_decrypted = cipher.decrypt(data)
    return text_decrypted
key = 'htsii__sht_eek.y'
m = AES_Decrypt(key, c).decode().replace('\x03','') # 去除最后的填充
print(m)
# LCTF{1t's_rea1ly_an_ea3y_ap4}
```

当然，有了key和密文，用cyberchef工具解密更快，解密的链接如下：
[AES解密](https://gchq.github.io/CyberChef/#recipe=AES_Decrypt%28%7B'option':'UTF8','string':'htsii__sht_eek.y'%7D,%7B'option':'Hex','string':''%7D,'ECB','Hex','Raw',%7B'option':'Hex','string':''%7D%29&input=MTVhM2JjYTI1Njc1ZWRiY2E0MjEzMjc2MTAwZDAxZjFmMzAzMDQ2N2VlNTExZTQ0MzZhMzJjZTk1ZDYyMDUzYg)

flag 为：LCTF{1t's_rea1ly_an_ea3y_ap4}

不敢相信自己的眼睛，竟然是really easy...

## 总结

花了几天的工夫，总算把4个小时的入门课程消化完了，
由于水平太菜，所以复盘的比较详细，方便过几天忘记了再回来查。
