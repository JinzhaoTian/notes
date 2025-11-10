![](../imgs/DesignPattern-Iterator.png)

#### 原理

遍历集合的接口，


#### 优点

1. 引入Iterator之后，可以将遍历与实现分离开来。

#### 缺点

1. 类太多


#### 示例

1. 分别创建 Iterator 接口和 Aggregate 接口：
```C++
template<class T>
class Iterator {
public:
	virtual bool hasNext() = 0;
	virtual T next() = 0;
}
```

```C++
template<class T>
class Aggregate {
public:
	virtual Iterator<T> iterator() = 0;
};
```

2. 实现具体的集合类和迭代器类：
```C++
class BookShelfIterator : public Iterator<Book> {
private:
	BookShelf bookShelf;
	int index;
public:
	bool hasNext() {
		if (index < bookShelf.getLength()) {
			return true;
		} else {
			return false;
		}
	}
	Book next() {
		Book book = bookShelf.getBookAt(index);
		index++;
		return book;
	}
};
```


```C++
class BookShelf : public Aggregate<Book> {
private:
	Book[] books;
	int length;
public:
	Book getBookAt(int index) {
		return books[index];
	}
	int getLength() {
		return length;
	}
	Iterator<Book>* iterator() {
		return new BookShelfIterator(this);
	}
};
```


3. 具体使用：
```C++
{
	Aggregate<Book>* bookShelf = new BookShelf();
	// 添加书本...
	Iterator<Book>* it = bookShelf -> iterator();
	while(it -> hasNext()) {
		Book book = it.next();
		// 处理
	}
}
```

