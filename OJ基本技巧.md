# OJ基本技巧记录

# OJ平台代码基本格式

```
import java.util.Scanner;

/**
 * Created by ml on 2017/9/11.
 */
public class Main {
    public static void main(String[] args) {
        Scanner in = new Scanner(System.in);
        while (in.hasNext()) {
            String n = in.nextLine();//读一个字符串
            int m = in.nextInt();//读一个整数
            long l = in.nextLong();//读一个长整数
            double t = in.nextDouble();//读一个浮点数
            System.out.println(cal(n));//具体处理逻辑
        }
        in.close();
    }
}
```

# OJ输入输出技巧


1. 输入数据有多行，第一行是一个整数n，表示测试实例的个数，后面跟着n行，每行包括一个由字母和数字组成的字符串。
```
Scanner in = new Scanner(System.in);
int n = in.nextInt();
//可以创建数组
int[] n = new int[n];
String[] str = new String[n];
for(int i=0;i<n;i++) {
    n[i] = in.nextInt();
    String str = in.nextLine();
}
```

2. 读入字符串的分割

例如：Input输入数据有多组，每组占一行，数据格式为YYYY/MM/DD组成 

```
Scanner in = new Scanner(System.in);
while (in.hasNext()) {
    String str =in.nextLine();
    String[] date = str.split("/");
    int y = Integer.parseInt(date[0]);
    int m = Integer.parseInt(date[1]);
    int d = Integer.parseInt(date[2]);
}
in.close();
```

3. 不同类型相互转换

- int转为String：
    - String s = Integer.toString(i);
    - String s = String.valueOf(i);
    - String s = "" + i;
    - 三种效率排序：Integer.toString(int i)   >   String.valueOf(int i)   >  i+"";    

- String转为int
    - int i = Integer.parseInt(s);
    - int i = Integer.valueOf(s).intValue()
    - 这两种都会抛出异常（NumberFormatException）


4. Java输出

- System.out.print(); //输出不换行

- System.out.println(); //输出换行 

- System.out.format(); 
    - double d = 345.678;
    - System.out.printf("%9.2f", d);// "9.2"中的9表示输出的长度，2表示小数点后的位数。
    - System.out.printf("%+-9.3f", d);// "+-"表示输出的数带正负号且左对齐。

- System.out.printf(); //与上相同

- DecimalFormat。DecimalFormat 类主要靠 # 和 0 两种占位符号来指定数字长度。0 表示如果位数不足则以 0 填充，# 表示只要有可能就把数字拉上这个位置。
    - NumberFormat formatter = new DecimalFormat( "0.00");   
    - String s = formatter.format(-.567);              //   -0.57   
    - System.out.println(s);  

5. 矩阵如何按行输出

```
//输出int[n][m]，判断是否是每一行的最后一个，输出换行。
for(int i = 0; i < n ; i++) {
    for(int j = 0; j < m; j++>) {
        if(j == m - 1){
            system.out.println(i);
        } else {
            system.out.print(i + " ");
    }
}
```

6. 不定长度类型

- 对于整形等数据在处理中不能确定长度的情况，使用ArrayList<Integer>等形式来存储数据。
- 对于char或者String类型可以使用StingBuffer等形式来存储。


7. 集合遍历

- 数组以及List：遍历
```
List<String> list = new ArrayList<String>();
for (int i = 0; i < list.size(); i++) {
    System.out.println(list.get(i));
}

```

- Map类
```
Map<String, String> map = new HashMap<String, String>();

//1
Iterator<Map.Entry<String, String>> it = map.entrySet().iterator();
while (it.hasNext()) {
    Map.Entry<String, String> entry = it.next();
    System.out.println("key= " + entry.getKey() + " and value= " + entry.getValue());
}
//2
for (Map.Entry<String, String> entry : map.entrySet()) {
    entry.getKey();
}
//3
for (String key : map.keySet()) {
    String value = (String) map.get(key);
}

```

- 传统的for循环遍历，基于计数器。遍历整个集合的平均时间复杂度为O(n^2)。
- 迭代器遍历，Iterator。
- foreach循环遍历。实现原理同Iterator。

8. 排序
- 如果要对数组排序，请使用java.util.Arrays.sort()方法。

- 如果对list排序，请使用java.util.Collections.sort()方法。

- 自定义排序：对于要排序的类实现Comparable接口。
```
class S1 implements Comparable{  
    int x;  
    int y;  
    S1(int x, int y){  
        this.x = x;  
        this.y = y;  
    }  
    //实现排序方法。先比较x，如果相同比较y  
    @Override  
    public int compareTo(Object o) {  
        S1 obj = (S1) o;  
        if(x != obj.x)  
        {  
            return x - obj.x;  
        }  
        return y - obj.y;  
    }  
    //重写toStirng方法，改变println时的显示效果  
    public String toString(){  
        return "("+x+", "+y+")";  
    }  

```

# OJ常用函数

1. 不同类型长度：
    - 数组：int len = n.length;//int[] n = new int[10];

    - 字符串：int len = s.length();//String s = "hello";

    - List：int len = list.size();//List<String> list = new ArrayList<String>();

2. String类：
    - 字符串某一位置字符,charAt。
    ```
    String str = new String("asdfzxc");
    char ch = str.charAt(4);//ch = z
    ```

    - 转化为char数组。toCharArray方法。
    ```
    String str1 = new String("abc");
    char[] c = str1.toCharArray();
    ```

    - 判断相等。equals方法。
    ```
    String str1 = new String("abc");
    String str2 = new String("ABC");
    boolean c = str1.equals(str2);//c=false
    ```

    - 提取子串。substring方法：一个参数：该方法从beginIndex位置起，出剩余的字符作为一个新的字符串返回。两个参数：左闭右开原则[beginIndex,endIndex)，即[beginIndex,endIndex - 1]
    ```
    String str1 = new String("asdfzxc");
    String str2 = str1.substring(2);//str2 = "dfzxc"
    String str3 = str1.substring(2,5);//str3 = "dfz"
    ```

    - 字符串比较。compareTo方法是对字符串内容按字典顺序进行大小比较，通过返回的整数值指明当前字符串与参数字符串的大小关系。若当前对象比参数大则返回正整数，反之返回负整数，相等返回0。
    ```
    String str1 = new String("abc");
    String str2 = new String("ABC");
    int a = str1.compareTo(str2);//a>0
    int b = str1.compareTo(str2);//b=0
    ```

    - 是否包含字符。contains方法判断参数s是否被包含在字符串中，并返回一个布尔类型的值.
    ```
    String str = "student";
    str.contains("stu");//true
    str.contains("ok");//false
    ```

    - 字符串字符替换。replace、replaceFirst、replaceAll方法。
    ```
    String str = "asdzxcasd";
    String str1 = str.replace('a','g');//str1 = "gsdzxcgsd"
    String str2 = str.replace("asd","fgh");//str2 = "fghzxcfgh"
    String str3 = str.replaceFirst("asd","fgh");//str3 = "fghzxcasd"
    String str4 = str.replaceAll("asd","fgh");//str4 = "fghzxcfgh"
    ```

    - 字符查找。indexOf、lastIndexOf方法。
    ```
    String str = "I am a good student";
    int a = str.indexOf('a');//a = 2
    int b = str.indexOf("good");//b = 7
    int c = str.indexOf("w",2);//c = -1
    int d = str.lastIndexOf("a");//d = 5
    int e = str.lastIndexOf("a",3);//e = 2
    ```
3. StringBuffer类

    - 添加。append方法。
    ```
    StringBuffer sb = new StringBuffer(“abc”);
    sb.append(true);//"abctrue"
    ```

    - 插入。insert方法。
    ```
    StringBuffer sb = new StringBuffer(“TestString”);
    sb.insert(4,false);//"TestfalseString"
    ```

    - 删除。deleteCharAt方法。
    ```
    StringBuffer sb = new StringBuffer(“Test”);
    sb. deleteCharAt(1);//"Tst"
    ```

    - 转换为字符串
    ```
    StringBuffer sb = new StringBuffer(“Test”);
    String s1 = sb.toString(); //StringBuffer转换为String
    ```

4. ArrayList类
    - 添加：add方法
    - 插入：insert方法
    - 删除：remove方法
    - 清空：clear方法
    - 包含：contains方法

# OJ常用算法

- 自己的初步阶段：理清思路，暴力求解，再逐步改进！