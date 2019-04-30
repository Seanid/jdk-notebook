# Arraylist的foreach过程中不能删除元素的原因

---

ArrayList是开发过程中常用的一种数据结构。通常在开发过程中，如果是对ArrayList进行遍历，可以使用foreach方式进行遍历，但是如果在遍历过程中会进行删除操作，通常会改用for(int i;i<list.size();i++)的方式进行操作，如果使用foreach，删除过程中会抛出异常

    
```
        ArrayList<Integer> list=new ArrayList();
        list.add(1);
        list.add(2);
        list.add(3);
        list.add(4);
        list.add(5);

        //foreach方式，删除到第二个的时候抛出ConcurrentModificationException
        /**
         * remove:1
         * Exception in thread "main" java.util.ConcurrentModificationException
         * 	at java.util.ArrayList$Itr.checkForComodification(ArrayList.java:819)
         * 	at java.util.ArrayList$Itr.next(ArrayList.java:791)
         */
        for(Integer i:list){
            list.remove(i);
            System.out.println("remove:"+i);
        }


        //正常删除
        for (int i=1;i<list.size();i++){
            list.remove(i);
            System.out.println("remove:"+i);
        }
```

为什么会这样？
```
//Iterator
 @SuppressWarnings("unchecked")
public E next() {
    checkForComodification();
    int i = cursor;
    if (i >= size)
        throw new NoSuchElementException();
    Object[] elementData = ArrayList.this.elementData;
    if (i >= elementData.length)
        throw new ConcurrentModificationException();
    cursor = i + 1;
    return (E) elementData[lastRet = i];
}
```

```
//Iterator
final void checkForComodification() {
	if (modCount != expectedModCount)
		throw new ConcurrentModificationException();
}
```

```
//ArrayList
public boolean remove(Object o) {
	if (o == null) {
		for (int index = 0; index < size; index++)
			if (elementData[index] == null) {
				fastRemove(index);
				return true;
			}
	} else {
		for (int index = 0; index < size; index++)
			if (o.equals(elementData[index])) {
				fastRemove(index);
				return true;
			}
	}
	return false;
}

private void fastRemove(int index) {
	modCount++;
	int numMoved = size - index - 1;
	if (numMoved > 0)
		System.arraycopy(elementData, index+1, elementData, index,
						 numMoved);
	elementData[--size] = null; // Let gc do its work
}
```


通过异常定位到jdk源码，可以知道foreach其实是使用迭代器的方式获取ArrayList中存储的对象，而在获取的时候，迭代器内部会进行一次修改次数的判断，如果判断失败，就会抛出异常。其中modCount是在ArrayList对象中声明定义的，而expectedModCount是在迭代器内部声明定义的。在调用list.remove(e)的时候，arraylist只对modCount进行了自增，由此导致了expectedModCount与modCount不同步，所以抛出了异常。之所以可以删除第一个，是因为删除第一个的时候modCount和expectedModCount还是同步的。