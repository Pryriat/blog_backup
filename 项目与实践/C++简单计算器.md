# 说明 
- 去年做的小玩意...实现了一些简单计算的功能，大致思想是将输入的算式作为字符串处理，每个子项为double类型。加入大数计算估计要推倒重做了233333.....欢迎讨论。

```cpp
//calculator.h
#include<iostream>
#include<string>
#include<vector>
using namespace std;
class num
{
private:
	double data;
	char g;
	friend void operator*(double a, num x);
public:
	void operator+ (const num &a)
	{
		data += a.data;
	}
	void operator=(double a)
	{
		data = a;
	}
	void operator-(const num &a)
	{
		data -= a.data;
	}
	void operator*(const num &a)
	{
		data *= a.data;
	}
	void operator*(double a)
	{
		data *= a;
	}
	void operator/(const num &a)
	{
		data /= a.data;
	}
	void print()
	{
		cout << data << endl;
	}
	num(double a)
	{
		data = a;
	}
	num()
	{
		cout << "error!";
	}
	void err()
	{
		g = '!';
	}
	char erro()
	{
		return g;
	}
	double nums()
	{
		return data;
	}
};
void operator*(double a, num x)
{
	x.data *= a;
}
string cindata();
void analyze(string &a);
vector<num> numbers(string &a);
class brancket
{
	public:
		vector<int> position;
		int times;
		brancket(vector<int> a, int n)
		{
			this->position = a;
			this->times = n;
		}
		brancket()
		{
			this->times = 0;
			vector<int>a;
			this->position = a;
		}
};
class erro_info
{
public:
	int err_num;
	char position1;
	char position2;
	char position3;
	int right_num;
	int left_num;
	erro_info(int a = 0, char b = ' ', char c = ' ', char d = ' ', int e = 0, int f = 0) : err_num(a), position1(b), \
		position2(c), position3(d), left_num(e), right_num(f) {}
};
```

```cpp
//calculator.cpp
#include<iostream>
#include<vector>
#include<fstream>
#include<conio.h>
#include<stdio.h>
#include"标头.h"
using std::cin;
using std::cout;
using std::endl;
using std::vector;
using std::ofstream;
using std::ifstream;
brancket test;
int menu()
{
	int choice;
	cout << "计算器ver0.02"<<endl;
	cout << "功能选择：1、软件说明 2、开始计算";
	cin >> choice;
	if (!cin)
		return 4;
	return choice;
}
string cindata()
{
	string equation;
	cin >> equation;
	return equation;
}
inline void pow(num& x, num& y)
{
	double time = y.nums();
	double temp = x.nums();
	if (time > 0)
		for (int z = 1; z < (int)time; z++)
			x * temp;
	else if (time == 0)
		x = time + 1.0;
	else if (time < 0)
	{
		for (int z = 1; z < -(int)time; z++)
			x*temp;
		temp = 1 / x.nums();
		x = temp;
	}
}
void analyze(string &a)
{
	int n = 0, num = 0, sign = 0, left_bracket = 0, right_bracket = 0;
	while (a[n] != '\0')
	{
		if (a[n] >= '0'&&a[n] <= '9')
		{
			if ((a[n + 1] >= '0'&&a[n + 1] <= '9') || a[n + 1] == '.')
			{
				n++;
				continue;
			}
			num++;
			n++;
		}
		else if (a[n] == '+' || a[n] == '-' || a[n] == '*' || a[n] == '/' || a[n] == '=' || a[n] == '^')
		{
			if (a[n + 1] == '^'|| a[n + 1] == '+'|| a[n + 1] == '-'|| a[n + 1] == '*'|| a[n + 1] == '/'||a[n+1]=='=')
			{
				throw erro_info(7, a[n - 1], a[n], a[n + 1]);
			}
			sign++;
			n++;
		}
		else if (a[n] == '(')
			{
			if (a[n + 1] == '-')
				sign--;
			left_bracket++;
			n++;
		}
		else if (a[n] == ')')
		{
			right_bracket++;
			n++;
		}
		else if (a[n] == '.')
		{
			n++;
		}
		else
		{
			throw erro_info(8, a[n]);
		}
	}
	if (num - sign != 0)
		throw erro_info(1);
	if (a[n - 1] != '=')
		throw erro_info(2);
	if (left_bracket != right_bracket)
		throw erro_info(3, ' ', ' ', ' ', left_bracket, right_bracket);
}
vector<num> numbers(string &a)
{
	vector<num> equ;
	double tempdata,l_temp=0,g=1;
	int n = 0, t = 0,dots=0;
	bool h = 0;
	while (a[n] != '\0')
	{
		while (a[n] >= '0'&&a[n] <= '9')
		{
			if (t == 0 && a[n] != 0)
				tempdata = a[n] - '0';
			else
			{
				tempdata *= 10;
				if (g == -1)
					tempdata  -= a[n] - '0';
				else
					tempdata  += a[n] - '0';
			}
			if(n>=2)
				if (a[n - 1] == '-'&&a[n - 2] == '(')
				{
					g = -1;
					tempdata *= -1;
					a.erase(a.begin() + n - 2);
					a.erase(a.begin() + n - 2);
					n -= 2;
					for (int i = n;; i++)
						if (a[i] == ')')
						{
							a.erase(a.begin() + i);
							break;
						}
				}
			t++;
			n++;
		}
		if (a[n] == '.')
		{
			dots++;
			g /= 10;
			n++;
			while (a[n] >= '0'&&a[n] <= '9')
			{
				l_temp += (a[n] - '0') * g;
				g /= 10;
				n++;
			}
			if (a[n] == '.')
			{
				throw erro_info(4, a[n - 2], a[n - 1], a[n]);
			}
		}
		if (a[n] == '+' || (a[n] == '-'&&a[n-1]!='(') || a[n] == '*' || a[n] == '/' || a[n] == '='||a[n] == '^')
		{
			equ.push_back(tempdata+l_temp);
			tempdata = 0;
			if (h)
			{
				equ[0].err();
			}
			l_temp = 0;
			t = 0;
			g = 1;
			n++;
		}
		else
			n++;
	}
	return equ;
}
vector<char> signs(string a)
{
	vector<char> signs;
	int n = 0;
	while (a[n] != '\0')
	{
		if ((a[n] < '0' || a[n] > '9')&&a[n]!='='&&a[n]!='.')
		{
			signs.push_back(a[n]);
		}
		n++;
	}
	return signs;
}
brancket checksign(vector<char>a)
{
	vector<int> temp_data;
	int times = 0;
	int position = 0;
	for (position; position < a.size(); position++)
	{
		if (a[position] == '(')
		{
			times++;
			temp_data.push_back(position);
		}
	}
	brancket temp(temp_data, times);
	return temp;
}
vector<num> caculate(vector<num>numbers, vector<char>signs, brancket checksign)
{
	if (numbers.size() == 0)
	{
		throw erro_info(5);
	}
	for (int time = checksign.times; time > 0; time--)
	{
		int right_brancket = 0;
		int position = checksign.position[time - 1];
		int temp_int = position;
		for (int i = 0; i < position; i++)
			if (signs[i] == ')')
				right_brancket++;
		for (position; signs[position] != ')'; position++)
		{
			if (signs[position] == '^')
			{
					pow(numbers[position - time - right_brancket], numbers[position - time - right_brancket + 1]);
					int temp = position - time - right_brancket;
					numbers.erase(numbers.begin() + temp + 1);
					signs.erase(signs.begin() + position);
					position--;
					temp = 0;
			}
		}
		position = temp_int;
		for (position; signs[position] != ')'; position++)
		{
			if (signs[position] == '*')
			{
				numbers[position - time - right_brancket] * numbers[position - time - right_brancket + 1];
				int temp =  position - time - right_brancket;
				numbers.erase(numbers.begin() + temp + 1);
				signs.erase(signs.begin() + position);
				position--;
				temp = 0;
			}
			else if (signs[position] == '/')
			{
				if (numbers[position - time + 1 - right_brancket].nums() == 0)
				{
					throw erro_info(6, signs[position], ' ', ' ', numbers[position - time - right_brancket].nums(), \
						numbers[position - time + 1 - right_brancket].nums());
				}
				numbers[position - time + -right_brancket] / numbers[position - time - right_brancket + 1];
				int temp = position - time - right_brancket;
				numbers.erase(numbers.begin() + temp + 1 );
				signs.erase(signs.begin() + position);
				position--;
				temp = 0;
			}
		}
		position = temp_int;
		for (position; signs[position] != ')'; position++)
		{
			if (signs[position] == '+')
			{
				numbers[position - time - right_brancket] + numbers[position - time - right_brancket + 1];
				int temp = position - time - right_brancket;
				numbers.erase(numbers.begin() + temp + 1 );
				signs.erase(signs.begin() + position);
				position--;
				temp = 0;
			}
			else if (signs[position] == '-')
			{
				numbers[position - time - right_brancket] - numbers[position - time - right_brancket + 1];
				int temp = position - time - right_brancket;
				numbers.erase(numbers.begin() + temp + 1 );
				signs.erase(signs.begin() + position);
				position--;
				temp = 0;
			}
		}
		signs.erase(signs.begin() + position);
		signs.erase(signs.begin() + temp_int);
	}
	int position = 0;
	for (position = 0; position < signs.size(); position++)
	{
		if (signs[position] == '^')
		{
			pow(numbers[position], numbers[position + 1]);
			numbers.erase(numbers.begin() + position + 1);
			signs.erase(signs.begin() + position);
			position--;
		}
	}
	for (position = 0; position < signs.size(); position++)
	{
		if (signs[position] == '*')
		{
			numbers[position] * numbers[position + 1];
			numbers.erase(numbers.begin() + position + 1);
			signs.erase(signs.begin() + position);
			position--;
		}
		else if (signs[position] == '/')
		{
			if (numbers[position + 1].nums() == 0)
			{
				throw erro_info(6, signs[position], ' ', ' ', numbers[position].nums(), numbers[position + 1].nums());
			}
			numbers[position] / numbers[position + 1];
			numbers.erase(numbers.begin() + position + 1);
			signs.erase(signs.begin() + position);
			position--;
		}
	}
	for (position = 0; position < signs.size(); position++)
	{
		if (signs[position] == '-')
		{
			numbers[position] - numbers[position + 1];
			numbers.erase(numbers.begin() + position + 1);
			signs.erase(signs.begin() + position);
			position--;
		}
		else if (signs[position] == '+')
		{
			numbers[position] + numbers[position + 1];
			numbers.erase(numbers.begin() + position + 1);
			signs.erase(signs.begin() + position);
			position--;
		}
	}
	return numbers;
}
int main()
{
	setlocale(LC_ALL, "");
	int choice = menu();
	char c = '0';
	do
	{
		switch (choice)
		{
		case 1:
			cout << "计算器ver 0.02" << endl;
			cout << "包括功能：加减乘除、次方括号" << endl;
			cout << "输入算式，以等于号结束" << endl;
			cin.get();
			cout << "输入x开始计算，输入q退出程序";
			cin >> c;
			if (c == 'x');
			else
				return 0;
		case 2:
			do
			{
				try
				{
					system("cls");
					cout << "输入算式" << endl;
					string a = cindata();
					system("cls");
					string temp = a;
					analyze(a);
					vector<num> t = numbers(a);
					vector<char> p = signs(a);
					brancket check = checksign(p);
					t = caculate(t, p, check);
					cout << endl;
					cout << temp;
					vector<num>::iterator it;
					for (it = t.begin(); it != t.end(); it++)
						(*it).print();

					cin.get();
					cout << "输入x继续计算" << endl;
					cin >> c;
				}
				catch (erro_info& a)
				{
					switch (a.err_num)
					{
					case 1:
						cout << "运算符与算子个数不匹配";
						break;
					case 2:
						cout << "算式末尾缺少等号";
						break;
					case 3:
						cout << "括号不匹配，左括号" << a.left_num << "个，右括号" << a.right_num << "个";
						break;
					case 4:
						cout << "小数点位置错误或连用,错误子式为" << a.position1 << a.position2 << a.position3;
						break;
					case 5:
						cout << "有效算式长度为零";
						break;
					case 6:
						cout << "被除数为0，错误子式为" << a.left_num << a.position1 << a.right_num;
						break;
					case 7:
						cout << "不支持符号连用，错误子式为" << a.position1 << a.position2 << a.position3;
						break;
					case 8:
						cout << "非法输入,非法字符为" << a.position1;
						break;
					}
					cin.get();
					cin.get();
					cout << "\n" << "输入x继续计算  ";
					cin >> c;
				}
			}while (c == 'x');
			break;
		default:
			cout << "选项输入错误，请重新输入" << endl;
			cin >> choice;
			continue;
		}
	} while (c == 'x');
	return 0;
}
```

- 顺便，用QT给这玩意套了层皮，效果如下，有需要可以去https://github.com/Pryriat/Caculator 自取
  ![](http://ozhtfx691.bkt.clouddn.com//calculatorscreenshot.png)