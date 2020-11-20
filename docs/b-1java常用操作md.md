## 1.遍历一个HashMap

- 通过foreach方式遍历

  ```
  Map<Integer, Integer> map = new HashMap<Integer,Integer>();
  
  for (Map.Entry<Integer, Integer> entry : map.entrySet()) {
  	entry.getKey();
  	entry.getValue();
  }
  ```

- foreach迭代键值对的方式

  ```
  for(Integer key : map.keySet()){
  	key;
  }
  for (Integer value: map.values()) {
  	value;
  }
  ```

- 使用带泛型的迭代器进行遍历 

  ```
  Iterator<Map.Entry<Integer, Integer>> entries = map.entrySet().iterator();
  while (entries.hasNext()) {
  	Map.Entry<Integer, Integer> entry = entries.next();
  	entry.getKey()
  	entry.getValue();
  }
  ```

- 使用不带泛型的迭代器进行遍历

  ```
  Iterator<Map.Entry> entries = map.entrySet().iterator();
  while (entries.hasNext()) {
  	Map.Entry entry = (Map.Entry)entries.next();
  	(Integer)entry.getKey();
  	(Integer)entry.getValue();
  }
  ```

- Lambda表达式

  map.forEach((k, v) - > System.out.println(k, v));

### list

- **offer，add区别**：

  一些队列有大小限制，因此如果想在一个满的队列中加入一个新项，多出的项就会被拒绝。

  这时新的 offer 方法就可以起作用了。它不是对调用 add() 方法抛出一个 unchecked 异常，而只是得到由 offer() 返回的 false。

- **poll，remove区别**：

  remove() 和 poll() 方法都是从队列中删除第一个元素。remove() 的行为与 Collection 接口的版本相似，

  但是新的 poll() 方法在用空集合调用时不是抛出异常，只是返回 null。因此新的方法更适合容易出现异常条件的情况。

- **peek，element区别**：

  element() 和 peek() 用于在队列的头部查询元素。与 remove() 方法类似，在队列为空时， element() 抛出一个异常，而 peek() 返回 null

## 2.整数和字符串相互转换

1. int转String有三种方式

   (1)num + ""

   (2)String.valueOf(num)

   (3)Integer.toString(num)

2. String转int有两种方式

   (1)Integer.parseInt(str)

   (2)Integer.valueOf(str).intValue()