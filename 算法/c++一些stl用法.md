## map

``` c++
#include<map>
using namespace std;

map<string,int> a;

int main(){
	a["test"] = 2; //直接赋值
	for(map<string,int>::iterator i=a.begin();i!=a.end();i++){//map也有迭代器
		cout<<i->first<<" "<<i->second<<endl;
	}
	map<string,int>::iterator ii = a.find("test");//找有没有对应迭代器，没有的话会返回a.end();
	cout<<"find "<<ii->first<<" "<<ii->second<<endl;
    //对于map,迭代器的first是key值，second是value值
}
```

## set

用法和map差不多，find也是返回指针，insert直接插入value。注意set内部直接红黑树，可以直接用来操作动态排序。创建排序结构体，重写里面的operator()方法，然后定义set的时候传入就可以了。

除了创建排序结构体以外，直接在元素内部重载操作符也是可以的。

``` c++
#include<set>
using namespace std;

struct mycmp{
	bool operator()(int x,int y){
		return x>y;
	}
};

set<int,mycmp> b;
int main(){
	b.insert(456);
	b.insert(54);
	b.insert(111);
	for(set<int>::iterator i=b.begin();i!=b.end();i++){
		cout<<*i<<" ";
	}
	cout<<endl;
}
```

## priority_queue
内部本质是一个heap，每次可以弹出最优先的那个

有几个需要注意的地方，首先是front改成了top对比queue，然后是注意定义是要写三个参数<type,vector<type>,mycmp>,mycmp也是自定义结构体重写operator()函数，但是注意这里的跟sort里的自定义函数是相反的，返回false的数据排在前面。
``` c++
#include<queue>
using namespace std;

struct nodes{
	int value;
	bool operator<(nodes x){
		return value>x.value;
	}
};

struct mycmp{
	bool operator()(nodes x,nodes y){
		return x.value>y.value;
	}
};

priority_queue<nodes,vector<nodes>,mycmp> c;
int main(){
	nodes n1,n2,n3;
	n1.value = 9;
	n2.value = 1;
	n3.value = 8;
	c.push(n1);
	c.push(n2);
	c.push(n3);
	while(!c.empty()){
	cout<<"front: "<<c.top().value<<endl;
	c.pop();
	}
}
```
