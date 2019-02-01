---
title: 畅读源码系列之JDK（一）：String
comments: true
categories:
  - 技术结构升级之JDK源码汇
tags:
  - 畅读源码
abbrlink: 7e8c5b5a
date: 2018-08-10 08:34:00
---
【引言】String，在我们日常开发过程中，想必也是最常用到的一种类型，所以，这次的畅读源码系列的第一站就选择来读一读String这个我们熟悉但可能有些地方还很陌生的类吧！
<div align=center><img src="http://pm4hdun71.bkt.clouddn.com/img/2018/2018-08-10-01.jpg" width="500"/></div>
<!-- more -->

# 声明
**&emsp;&emsp;本系列的源码解读，也只是针对个人觉得比较重要的部分进行基本解读和分析，不可能对整个源代码包的所有文件面面俱到，所以只对有必要分析的部分加一些个人见解，非必要的部分在文中不做任何体现。**

# 类的定义

## String
```java
/**
 * 【Lin.C】：首先，从开篇的图和String类的源码可以看出几个相当明显的特性：
 *         - String是final的，这也就注定了它是不能被继承，final类中的所有成员方法都会被隐式地指定为final方法
 *         - 实现了Serializable：表示String是可序列化的
 *         - 实现了Comparable：表示String是可比较的
 *         - 实现了CharSequence：实际上CharSequence是一个接口，包括StringBuffer和StringBuilder也实现了CharSequence接口
 *         - CharSequence与String都能用于定义字符串，但CharSequence的值是可读可写序列，而String的值是只读序列
 */
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    ......
}
```

## CharSequence
```java
/**
 * 【Lin.C】：可读可写序列
 * @author Mike McCloskey
 * @since 1.4
 * @spec JSR-51
 */

public interface CharSequence {

    /**
     * 【Lin.C】：字符序列的长度
     */
    int length();

    /**
     * 【Lin.C】：取得某个位置上的char
     */
    char charAt(int index);

    /**
     * 【Lin.C】：截取
     */
    CharSequence subSequence(int start, int end);

    /**
     * 【Lin.C】：转成String
     */
    public String toString();
    
    // 【Lin.C】：还有一些default的方法（1.8新增的），这里就不列举了
}
```

# 成员变量
```java
    /**
     * 【Lin.C】：很明显，这个数组是用做基础的存储数据结构的；所以最终有可能后续的分析都需要结合数组的特性了
     */
    /** The value is used for character storage. */
    private final char value[];
    
    /**
     * 【Lin.C】：hash实际就是个散列，通常我们常见的用法就是在equals时必须要用到的hashcode方法
     */
    /** Cache the hash code for the string */
    private int hash; // Default to 0
```

# 构造方法
```java
    /**
     * 【Lin.C】：这个空参构造方法本身很简单，但要结合下面的另一个构造方法看就更明了了
     */
    public String() {
        // 可以理解为 new String("").value;
        this.value = "".value;
    }
    
    /**
     * 【Lin.C】：这里就有了初始化参数了，实际就是根据初始化参数给value赋值，同时给hash赋值
     */
    public String(String original) {
        this.value = original.value;
        this.hash = original.hash;
    }
    
    // 【Lin.C】：还有很多基于数组之类的构造方法，因为用的不多，这里就不一一列举了
```

# 常用方法

## charAt
```java
    /**
     * 【Lin.C】：前面了解了基本的几个属性，这个方法就很好理解了
     *        - 如果下标越界了，就抛个异常
     *        - 否则，就返回value数组对应index的那个char
     */
    public char charAt(int index) {
        if ((index < 0) || (index >= value.length)) {
            throw new StringIndexOutOfBoundsException(index);
        }
        return value[index];
    }
```

## compareTo
```java
    /**
     * 【Lin.C】：字符串比较方法
     *        - 如果下标越界了，就抛个异常
     *        - 否则，就返回value数组对应index的那个char
     */
     public int compareTo(String anotherString) {
        
        // 【Lin.C】：算出长度较短的一个String（只比较到这个长度）
        int len1 = value.length;
        int len2 = anotherString.value.length;
        int lim = Math.min(len1, len2);
        
        // 【Lin.C】：取两个String的底层value数组
        char v1[] = value;
        char v2[] = anotherString.value;
    
        // 【Lin.C】：逐位比较
        int k = 0;
        while (k < lim) {
            char c1 = v1[k];
            char c2 = v2[k];
            
            // 【Lin.C】：一旦出现不相等的位，则返回（原始串大则返回正数，原始串小则返回负数）
            if (c1 != c2) {
                return c1 - c2;
            }
            k++;
        }
        
        // 【Lin.C】：若前面都相等，则返回长度差（原始串长则返回正数，原始串短则返回负数，一样长则返回0）
        return len1 - len2;
    }
```

## concat
```java
    /**
     * 【Lin.C】：字符串连接方法（"cares".concat("s") returns "caress"）
     */
    public String concat(String str) {
        
        // 【Lin.C】：很简单的逻辑，追加的字符串若长度为0，则返回原String
        int otherLen = str.length();
        if (otherLen == 0) {
            return this;
        }
        int len = value.length;
        
        // 【Lin.C】：否则，扩容数组（大小为两个字符串的长度和），并将原数组的值复制写入新数组
        char buf[] = Arrays.copyOf(value, len + otherLen);
        
        // 【Lin.C】：把str追加到buf的后面
        str.getChars(buf, len);
        
        // 返回这个新的String（这个构造方法的实现：this.value = value; 后面那个true没有任何意义）
        return new String(buf, true);
    }
    
    /**
     * 【Lin.C】：将数组复制扩容，并将原数组的值复制写入新数组
     */
    public static char[] copyOf(char[] original, int newLength) {
        char[] copy = new char[newLength];
        System.arraycopy(original, 0, copy, 0, Math.min(original.length, newLength));
        return copy;
    }
    
    /**
     * 【Lin.C】：将value追加到从第0位开始复制到dst数组的dstBegin位置
     */
    void getChars(char dst[], int dstBegin) {
        System.arraycopy(value, 0, dst, dstBegin, value.length);
    }
    
    /**
     * 【Lin.C】：System提供的一个native方法；将一个数组的指定个数元素复制到另一个数组中
     *        - src、srcPos：被复制的数组和复制起始位置
     *        - dest、destPos：接收复制数组的目标数组和接收的起始位置
     *        - length：要复制的字符个数
     * 【补】：当dest未从尾部开始，则dest后面的字符被覆盖；当destPos+length>dest.length，则会抛出数组越界异常
     */
    public static native void arraycopy(Object src,  int  srcPos,
                                        Object dest, int destPos,
                                        int length);
    
    /**
     * Main参考
     * @param args
     */
    public static void main(String[] args) {

        // 假设有两个数组需要串联
        char[] value1 = "Cheng".toCharArray();
        char[] value2 = "Lin".toCharArray();

        int len1 = value1.length;
        int len2 = value2.length;

        // 第一步需要扩容一个能容纳原来两个数组的大数组
        char buf[] = Arrays.copyOf(value1, len1 + len2);

        // 然后将第二个数组追加复制到第一个数组的尾部
        System.arraycopy(value2, 0, buf, len1, len2);
        System.out.println("value1 = " + new String(value1));
        System.out.println("value2 = " +new String(value2));
        System.out.println("buf = " +new String(buf));
               
        /* 【结果】： 
        value1 = Cheng
        value2 = Lin
        buf = ChengLin
        */
    }

```

## contains & indexOf
```java
    /**
     * 【Lin.C】：这个方法就很简单，判断字串在父串内的index是否大于0
     */
    public boolean contains(CharSequence s) {
        return indexOf(s.toString()) > -1;
    }
    
    /**
     * 【Lin.C】：indexOf的参数拆分后调用下面一个比较复杂的indexOf方法实现定位
     */
    public int indexOf(String str, int fromIndex) {
        return indexOf(value, 0, value.length,
                str.value, 0, str.value.length, fromIndex);
    }
    
    /**
     * 【Lin.C】：实际由好几个重载的indexOf方法，但最终都是调用到下面的这个方法，所以看这个就够了
     * 
     * @param   source       the characters being searched.
     * @param   sourceOffset offset of the source string.
     * @param   sourceCount  count of the source string.
     * @param   target       the characters being searched for.
     * @param   targetOffset offset of the target string.
     * @param   targetCount  count of the target string.
     * @param   fromIndex    the index to begin searching from.
     */
    static int indexOf(char[] source, int sourceOffset, int sourceCount,
            char[] target, int targetOffset, int targetCount,
            int fromIndex) {
        if (fromIndex >= sourceCount) {
            return (targetCount == 0 ? sourceCount : -1);
        }
        if (fromIndex < 0) {
            fromIndex = 0;
        }
        if (targetCount == 0) {
            return fromIndex;
        }

        // 【Lin.C】：先找到第一个target字符
        char first = target[targetOffset];
        
        // 【Lin.C】：最大查找位置（如"Chenglin".indexOf("en"); sourceOffset=0,sourceCount=8,targetCount=2,则该值为6）
        // 【Lin.C】：也就是说剩余长度不够targetCount的长度时，后面没必要再检查了
        int max = sourceOffset + (sourceCount - targetCount);

        // 【Lin.C】：当第一个字符命中了，但后续没有命中的情况下，需要通过这个循环控制多次去匹配（i++往后偏移一位）
        for (int i = sourceOffset + fromIndex; i <= max; i++) {
        
            // 【Lin.C】：先找第一个字符，如果第一个都找不到，那后面就不用玩了
            /* Look for first character. */
            if (source[i] != first) {
                while (++i <= max && source[i] != first);
            }

            // 【Lin.C】：i<=max就表示第一个字符正常找到了
            /* Found first character, now look at the rest of v2 */
            if (i <= max) {
                int j = i + 1;
                int end = j + targetCount - 1;
                
                // 【Lin.C】：然后将target从第2个字符开始，挨个和source的随后字符比较，一旦出现不相等的情况，for结束
                for (int k = targetOffset + 1; j < end && source[j] == target[k]; j++, k++);
                
                // 【Lin.C】：最后确认命中的index是不是满足target的长度条件，满足则返回命中的位置了
                if (j == end) {
                    /* Found whole string. */
                    return i - sourceOffset;
                }
            }
        }
        return -1;
    }
    
    /**
     * 【Lin.C】：基于单个字符的index查找
     */
    public int indexOf(int ch, int fromIndex) {
        final int max = value.length;
        
        // 【Lin.C】：基本的参数校正和长度判断条件
        if (fromIndex < 0) {
            fromIndex = 0;
        } else if (fromIndex >= max) {
            // Note: fromIndex might be near -1>>>1.
            return -1;
        }

        // 【Lin.C】：必须在合法字符的要求范围内
        // The minimum value of a Unicode supplementary code point, constant(0x010000)
        if (ch < Character.MIN_SUPPLEMENTARY_CODE_POINT) {
            // handle most cases here (ch is a BMP code point or a
            // negative value (invalid code point))
            final char[] value = this.value;
            
            // 【Lin.C】：逐个查找，找到一个命中的就返回对应的index
            for (int i = fromIndex; i < max; i++) {
                if (value[i] == ch) {
                    return i;
                }
            }
            return -1;
        } else {
            return indexOfSupplementary(ch, fromIndex);
        }
    }
```

## startsWith & endsWith
```java
    /**
     * 【Lin.C】：endsWith本身是通过startsWith实现的，有点意思（实际就是控制偏移量为反向suffix的长度）
     */
    public boolean endsWith(String suffix) {
        return startsWith(suffix, value.length - suffix.value.length);
    }
    
    /**
     * 【Lin.C】：判断是否是以某个String开头的（可以有偏移量）
     */
    public boolean startsWith(String prefix, int toffset) {
        char ta[] = value;
        int to = toffset;
        char pa[] = prefix.value;
        int po = 0;
        int pc = prefix.value.length;
        // Note: toffset might be near -1>>>1.
        
        // 【Lin.C】：首先从长度上来避免进入循环判断，长度不够匹配肯定是false的
        if ((toffset < 0) || (toffset > value.length - pc)) {
            return false;
        }
        
        // 【Lin.C】：然后带着偏移量挨个字符比较，一旦发现有不相等的，立马返回false
        while (--pc >= 0) {
            if (ta[to++] != pa[po++]) {
                return false;
            }
        }
        
        // 【Lin.C】：走到这里表示prefix在原串里面完整命中了，就是true
        return true;
    }
```

## length
```java
    /**
     * 【Lin.C】：作为一个非常常用的方法，虽然实现简单，这里还是提一句吧
     */
    public int length() {
        return value.length;
    }
```

## replace
```java     
    /**
     * 【Lin.C】：不涉及正则表达式，单纯的数组位置替换
     */
    public String replace(char oldChar, char newChar) {
        ......
    }
    
    /**
     * 【Lin.C】：被替换的target不作为正则规则，只作为纯粹的字符串替换
     */    
    public String replace(CharSequence target, CharSequence replacement) {
        // 【Lin.C】：指定Pattern.LITERAL标志后，输入字符串就会作为字面值字符序列来对待（不作为正则表达式编译）
        return Pattern.compile(target.toString(), Pattern.LITERAL).matcher(
                this).replaceAll(Matcher.quoteReplacement(replacement.toString()));
    }
    
    /**
     * 【Lin.C】：被替换的regex作为正则规则，通过Pattern进行compile后再进行替换；全文替换
     */    
    public String replaceAll(String regex, String replacement) {
        return Pattern.compile(regex).matcher(this).replaceAll(replacement);
    }
    
    /**
     * 【Lin.C】：被替换的regex作为正则规则，通过Pattern进行compile后再进行替换；只替换第一个
     */    
    public String replaceFirst(String regex, String replacement) {
        return Pattern.compile(regex).matcher(this).replaceFirst(replacement);
    }
    
    /**
     * Main
     * @param args
     */
    public static void main(String[] args) {
        String str = "chenglin.test.txt";
        System.out.println(str.replace(".", "#"));
        System.out.println(str.replaceAll(".", "#"));
        System.out.println(str.replaceFirst(".", "#"));
        
        /* 【结果】：
        chenglin#test#txt
        #################
        #henglin.test.txt
        */
    }
```

## hashCode
&emsp;&emsp;之所以使用 31， 是因为他是一个奇素数。如果乘数是偶数，并且乘法溢出的话，信息就会丢失，因为与2相乘等价于移位运算（低位补0）。使用素数的好处并不很明显，但是习惯上使用素数来计算散列结果。 31 有个很好的性能，即用移位和减法来代替乘法，可以得到更好的性能： 31 * i == (i << 5） - i， 现代的 VM 可以自动完成这种优化。
```java
    /**
     * 【Lin.C】：一个涉及到equals和散列的方法;使用 String 的 char 数组的数字每次乘以 31 再叠加最后返回
     *            因此，每个不同的字符串，返回的 hashCode 肯定不一样
     */
    public int hashCode() {
        int h = hash;
        if (h == 0 && value.length > 0) {
            char val[] = value;

            for (int i = 0; i < value.length; i++) {
                h = 31 * h + val[i];
            }
            hash = h;
        }
        return h;
    }
```

## substring
```java
    /**
     * 【Lin.C】：代码本身很简单，但是需要注意若产生切割会生成一个新的String
     */
    public String substring(int beginIndex, int endIndex) {
        if (beginIndex < 0) {
            throw new StringIndexOutOfBoundsException(beginIndex);
        }
        if (endIndex > value.length) {
            throw new StringIndexOutOfBoundsException(endIndex);
        }
        int subLen = endIndex - beginIndex;
        if (subLen < 0) {
            throw new StringIndexOutOfBoundsException(subLen);
        }
        return ((beginIndex == 0) && (endIndex == value.length)) ? this
                : new String(value, beginIndex, subLen);
    }
```

## trim
```java
    /**
     * 【Lin.C】：把首尾为空的char逐个剔除，然后substring生成一个新的String
     */
    public String trim() {
        int len = value.length;
        int st = 0;
        char[] val = value;    /* avoid getfield opcode */

        while ((st < len) && (val[st] <= ' ')) {
            st++;
        }
        while ((st < len) && (val[len - 1] <= ' ')) {
            len--;
        }
        return ((st > 0) || (len < value.length)) ? substring(st, len) : this;
    }
```