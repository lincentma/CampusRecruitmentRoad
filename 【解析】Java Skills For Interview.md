# 【解析】Java Skills For Interview

> 从网上偶得之，把每个函数弄明白，源码是怎么写的，为什么这么写。



## 代码

```java

//import不要使用通配符，需要import哪一个就import哪一个。
import java.util.*;
 
public class JavaSkillsForInterview {
    public static void main(String[] args) {
        
        // String
        String s = "abc";
        s.charAt(0);//返回指定索引处的 char 值。
        s.length();//返回此字符串的长度。
        s.substring(0, 1);//返回字符串的子字符串。beginIndex - 起始索引（包括），endIndex - 结束索引（不包括）。左闭右开。
        s.substring(1);//返回beginIndex - 起始索引（包括）到字符串末尾的子字符串。
        s.equals("b");//将此字符串与指定的对象比较。
        s = s.trim();//返回字符串的副本，忽略前导空白和尾部空白。用于删除字符串的头尾空白符。
        s.indexOf("a");//返回指定字符在此字符串中第一次出现处的索引。如果此字符串中没有这样的字符，则返回-1。
        s.indexOf("a", 1);//返回在此字符串中第一次出现指定字符处的索引，从指定的索引开始搜索。
        s.lastIndexOf("a");//返回指定字符在此字符串中最后一次出现处的索引。
        s.lastindexOf("a", 1);//返回指定字符在此字符串中最后一次出现处的索引，从指定的索引处开始进行反向搜索。
        s.toCharArray();//【常用】将此字符串转换为一个新的字符数组。
        Integer.valueOf(s); // returns an Integer object，valueOf会返回一个Integer（整型）对象
        //【注意】Integer类有一个静态缓存，存储了256个特殊的Integer对象——每个对象分别对应`-128 和127之间的一个值。
        // Integer.valueOf("127")==Integer.valueOf("127");【true】
        // Integer.valueOf("128")==Integer.valueOf("128");【false】
        Integer.parseInt(s); // returns an int primitive， 将形参s转化为整数。Interger.parseInt("1")=1;
        String.valueOf(s); // integer to string 返回指定参数的字符串表示形式。

        // StringBuilder
        StringBuilder sb = new StringBuilder();
        sb.append("a");//将指定的字符串追加到此字符序列。
        sb.insert(0, "a"); //在index指示的字符之前插入指定的字符串。
        sb.deleteCharAt(sb.length() - 1); //在这个序列中的指定位置，此方法将删除字符。	
        sb.reverse(); //此方法会导致此字符序列被替换为该序列的反转序列。
        sb.toString(); //该方法返回一个字符串，它表示这个序列中的数据。
        //String 长度大小不可变
        //StringBuffer 和 StringBuilder 长度可变
        //StringBuffer 线程安全 StringBuilder 线程不安全
        //StringBuilder 速度快
        
        // Array
        int[] a = new int[10];//初始化方式1
        char[] b = {'a', 'b'};//初始化方式2
        int[][] c = new int[10][10];//二维数组
        int m = a.length;//数组长度
        int n = c[0].length;//二维数组的列数
        int max = Integer.MAX_VALUE; //MAX_VALUE = 0x7fffffff （Java语言规范规定int型为4字节）
        int min = Integer.MIN_VALUE; //MIN_VALUE = 0x80000000
        Arrays.sort(a);//数组升序排序	（import java.util.Arrays;）int[]，double[]，char[]等基数据类型的数组，只提供了默认的升序排列，没有提供相应的降序排列方法。
        for (int i = 0; i < c.length; i++) {
            System.out.println(c[i]);
        }// 遍历数组输出元素
        
        // List
        //List是一个接口，而ArrayList是一个类。 ArrayList继承并实现了List。 
        List<Integer> list = new ArrayList<Integer>();//List初始化
        ArrayList<Integer> list1 = new ArrayList<Integer>(); //ArrayList初始化
        List<List<Integer>> list2 = new ArrayList<List<Integer>>(); //而为List
        list.add(0);//指定元素E追加到列表的末尾。此方法返回true。
        list.add(0, 1);//方法将指定的元素E在此列表中的指定位置。此方法不返回任何值。
        list.addAll(list1);//方法会将所有指定集合中的元素添加到此列表的结尾。
        list.get(0);//此方法返回在此列表中的指定位置的元素。
        list.size();//此方法返回此列表中的元素数。
        list.remove(list.size() - 1);//此方法返回从列表中移除的元素。
        //Collections是一个类而Collection是一个接口。
        Collections.sort(list);//方法用于指定列表按升序进行排序，根据其元素的自然顺序。
        Collections.sort(list, Collections.reverseOrder());//反序排列。
        //自定义排序。根据Collections.sort重载方法来实现。
        Collections.sort(list, new Comparator<Integer>() {
            @Override
            public int compare(Integer o1, Integer o2) {
                return o1 - o2;// 0->1
                // return o2 - o1; 1->0
            }
        });
        
        // Stack
        // import java.util.Stack;  
        Stack<Integer> stack = new Stack<Integer>();
        stack.push(0);//把项item压入栈顶，返回item参数。
        stack.pop();//返回位于堆栈顶部的元素，在这个过程中除去它。
        stack.peek();//返回到堆栈顶部的元素，但不会将其删除。
        stack.isEmpty();//栈是否为空。空返回true，否则false。
        // isEmpty() 和 empty()的区别：命名区别。
        // For example, it was named empty() in original class but was named isEmpty() of Collection interface.
        stack.size(); //栈中元素的个数。
        stack.search("code"); //此方法返回从1开始的位置，一个对象在栈中。栈顶位置为1。
        
        // Queue add ‐‐‐‐‐‐> remove, peek
        // import java.util.Queue;  import java.util.LinkedList;
        Queue<Integer> q = new LinkedList<Integer>();
        q.add(0);//增加一个元索。
        q.remove();//移除并返回队列头部的元素。
        q.peek();//返回队列头部的元素。
        //Queue使用也可以offer()来加入元素，使用poll()来获取并移出元素。
        q.isEmpty();//返回队列是否为空。
        q.size();//返回队列中元素的个数。
        
        // HashMap
        // import java.util.HashMap;
        HashMap<Character, Integer> map = new HashMap<Character, Integer>();//初始化。
        map.put('c', 1);//关联与此映射中的指定键指定的值。添加映射。
        map.get('c');//返回指定键映射在此标识哈希映射，或者null，如果映射不包含此键的值。
        if (map.containsKey('c')) {//如果此映射包含指定键的映射关系返回true。
        }
        if (map.containsValue(1)) {//如果此映射一个或多个键映射到指定值返回true。
        }
        for (Character d : map.keySet()) {//遍历Hashmap的键集合。Set keySet()返回此映射中包含的键的set视图。
        }
        for (Integer i : map.values()) {//遍历Hashmap的值集合。Collection values()返回此映射中包含的值的collection视图。
        }
        map.isEmpty();//如果此映射不包含键 - 值映射关系返回true。
        map.size();//返回键 - 值映射关系在这个映射中的数量。
        
        // HashSet
        // HashSet借助HashMap来实现的，利用HashMap中Key的唯一性，来保证HashSet中不出现重复值。
        // HashMap中的Key是根据对象的hashCode() 和 euqals()来判断是否唯一的。
        // import java.util.HashSet;
        HashSet<Integer> set = new HashSet<Integer>();//初始化。
        set.add(0);//将指定的元素添加到此集合，如果它是不存在的。
        set.remove(0);//从集合中删除指定的元素（如果存在）。
        if (set.contains(0)) {//如果此set包含指定的元素，则返回true。
        }
        set.isEmpty();//返回true如果此set不包含任何元素。
        set.size();//返回元素的数目（它的基数）。
        
        // mini heap
        // import java.util.PriorityQueue;
        PriorityQueue<Integer> pq = new PriorityQueue<Integer>();//初始化。
        pq.add(0);//该方法调用返回true(所指定的Collection.add(E))
        pq.offer(0);//该方法调用返回true(所指定的Queue.offer(E))
        pq.remove();//该方法调用返回true，如果此队列由于调用而更改的结果。
        pq.peek();//在方法调用返回此队列的头部，或null，如果此队列为空。但它不会将其删除。
        pq.poll();//方法用于检索并移除此队列的头，则返回null，如果此队列为空。	
        pq.isEmpty();//判读是否为空。
        pq.size();//在方法调用返回的元素在此集合数
        while (!pq.isEmpty()) {
        }
    }

}
```



## 阿里巴巴Java开发规范

[阿里巴巴Java开发规范]: http://orj5hqpmw.bkt.clouddn.com/4ce9918d7f5a082177a8a2741d0ab3ce.pdf