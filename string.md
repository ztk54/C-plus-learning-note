# constructor
以下为常见构造：
1. `string (const string& str)`拷贝一个其他的string类型
2. `string(const string& str, size_t pos, size_t len = npos)`从str字符串的pos位置开始，向后拷贝len个字符
3. `string (size_t n, char c)`构造n个字母c
4. `string (const char* s)`拷贝字符串常量
# getline和流输入
我们使用cin>>时候默认遇到空格或换行符就会停止输入，但是有时候我们想要保留空格或者使用我们指定的符号作为终止，这时候就用到getline
```cpp

istream& getline (istream& is, string& str, char delim);
istream& getline (istream& is, string& str);
```
delim是自定义结束符
# Element access
1. 使用[]操作符，和数组使用类似
2. at操作符，`string topic;topic.at(2)`与1的区别为这个会抛异常
3. front和back取首尾字符
# Iterators迭代器与遍历
1. []＋下标遍历
```cpp
int count = 0;
	//下标＋[]
	for (size_t i = 0;i < preview.size();i++)
	{
		if (preview[i] == ' ')
		{
			++count;
			preview[i] = '_';
		}
	}
```
2. 范围for
```cpp
//auto范围for
for (char& ch : preview)//这里ch加引用修改原值
{
	if (ch == ' ')
	{
		++count;
		ch = '_';
	}
}
```
3. 迭代器
```cpp
//迭代器
for (auto it = preview.begin();it != preview.end();++it)
{
	if (*it == ' ')
	{
		++count;
		*it = '_';
	}
}
```