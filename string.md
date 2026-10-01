# std::string 速查

`#include <string>`。`std::string` 管理一段可修改的文本；拷贝后两个字符串的内容可分别修改。

## 1. 构造与状态

```cpp
string a;                    // 空串
string b = "Date class";     // 从字符串字面量构造
string c(b);                 // 拷贝
string d(b, 5, 5);           // 从位置 5 取最多 5 个字符：class
string line(20, '-');        // 20 个 '-'
```

`size()` 返回长度，`empty()` 判断是否为空；空格也算一个字符。截取构造的起点超过原串长度会抛出 `out_of_range`。

## 2. 输入

`cin >> s` 读到空白字符就停；`getline(cin, s)` 读一整行，包括中间的空格，但不把结尾换行存入 `s`。`getline(cin, s, delim)` 可指定结束符，结束符会被读取但不存入 `s`。

若前面用了 `cin >>`，行尾换行通常仍留在输入流里；紧接着调用 `getline` 可能先读到空串。需要按输入格式先处理这一行的剩余内容。

## 3. 访问与遍历

- `s[i]` 访问下标；`s.at(i)` 会对越界下标抛异常。
- `s.front()` / `s.back()` 取首尾字符；空串时不要调用。
- 下标循环适合需要位置的场景；范围 `for (char& ch : s)` 可直接修改字符；迭代器从 `begin()` 走到 `end()`，`end()` 本身不指向字符。
- 同一个字符串上连续运行三种“替换空格”循环，第一种已经替换的空格，后两种就不会再遇到；要比较三种方法，可给它们各用一份副本。

## 4. 运算符与比较

- `=` 赋值，`+` 生成拼接结果，`+=` 在原字符串后面追加。
- `==` / `!=` 判断内容是否相同；`<` / `>` 做字典序比较，适合按文本排序，不等于比较长度。
- `string copy = original;` 后修改 `copy`，`original` 不会跟着改变。

## 5. 修改内容

| 操作 | 用途 |
|---|---|
| `push_back(c)` | 尾部加一个字符 |
| `append(text)` | 尾部追加文本 |
| `insert(pos, text)` | 在位置 `pos` 前插入文本 |
| `replace(pos, n, text)` | 从 `pos` 起替换最多 `n` 个旧字符 |
| `erase(pos, n)` | 从 `pos` 起删除最多 `n` 个字符；省略 `n` 则删到末尾 |
| `clear()` | 清空内容；之后 `empty()` 为 `true` |

插入、删除或替换后，后续位置可能改变；要用**修改后的字符串**重新计算位置。之前保存的迭代器或字符引用也可能失效。

## 6. 查找与截取

```cpp
size_t pos = s.find("Date");
if (pos == string::npos) cout << "未找到";
else cout << "从位置 " << pos << " 开始";
```

- `find(text)` 找第一次出现的位置；`find(text, start)` 从指定位置继续找；`rfind(text)` 找最后一次出现的位置。
- `npos` 表示找不到；位置 `0` 是有效结果，不等于“没找到”。使用位置前先检查 `npos`。
- `substr(pos, n)` 从 `pos` 起复制最多 `n` 个字符，返回新字符串；第二个参数是**长度**，省略它则取到末尾。`pos == size()` 得到空串，`pos > size()` 抛异常。

## 动态存储
1. `resize(n)`把字符串长度改为n，`resize(n+2,'+')`可以追加2个字符+
2. `capacity()`：目前准备了多少空间来容纳字符
3. `reserve(n)`：提前准备空间，不添加字符
4. `size()`：现在有多少个字符

## C 接口与内嵌零字符

`c_str()` 返回以 `\0` 结尾的 `const char*`，供需要 C 风格字符串的接口使用；`cout << std::string` 本身就能输出。字符串中间也可以含 `\0`：`size()` 仍计入它，`strlen(c_str())` 只数到第一个 `\0`。修改或扩容后，之前取得的指针可能失效，需要重新获取。

## stoi

```cpp
	string s;
	cout << "请以主题|次数的格式输入" << endl;
	getline(cin, s);
	size_t pos = s.find('|');
	if (pos == string::npos)
	{
		cout << "输入不符合要求" << endl;
	}
	else
	{
		size_t used = 0;
		string count = s.substr(pos + 1);
		s.erase(pos + 1);
		int tmp1 = stoi(count,&used);
		if (used != count.size())
		{
			cout << "次数只能为整数" << endl;
		}
		else
		{
			tmp1++;
			string tmp2 = to_string(tmp1);
			s += tmp2;
			cout << s << endl;
		}
	}
```

## 这次手写 `string`：知识点与易错点

- **长度与容量**：`size` 是当前有效字符数，`capacity` 是不用扩容时最多能放的有效字符数。教学版至少申请 `capacity + 1` 格，多的一格放结尾 `\0`；始终保持 `size <= capacity`、`data[size] == '\0'`。
- **空串也要有有效存储**：本版选择 `new char[1]`，写入 `\0`。`nullptr` 没有第 0 格，不能执行 `ptr[0] = '\0'`。
- **申请与释放**：`new[]` 配 `delete[]`。`char* q = p` 只复制地址，不复制数组；含原始指针的类若直接用默认拷贝，两个对象会共用数组，可能重复释放。写好深拷贝前先禁用拷贝。
- **复制和扩容**：复制 `size + 1` 格才能保留末尾 `\0`。扩容时先申请新数组、复制旧内容，再释放旧数组；`reserve` 不改变 `size`。容量翻倍是一种策略，并非固定规则。
- **命名空间与输出**：`<cstring>` 提供 `std::strlen`、`std::memcpy`；`<iostream>` 提供 `std::ostream`、`std::cout`。进入 `ztk` 命名空间后仍需写 `std::`。`cout << 自定义对象` 需要自己的 `operator<<`；初期可用 `cout << 对象.c_str()` 测试。
- **小工程验证**：头文件放声明，`.cpp` 放定义，测试文件放 `main`。编译通过后实际输出空串和普通串，并检查末尾 `\0`；成员初始化的实际顺序由声明顺序决定。

## 后续实践

`std::string` 常用接口和学习记录综合任务已通过。当前练教学版 `ztk::string` 的内存管理、扩容、追加与深拷贝；图片中的其他接口按需挑选。`stoi` 的空次数、超大数字异常按计划延后。

接口目录：[cplusplus.com：std::string](https://cplusplus.com/reference/string/string/)。
