# install
`boost_1_61_0.tar.gz`下载并解压, 执行以下命令:
```shell
./bootstrap.sh
./b2

```
* include paths: path/to/boost_1_61_0
* library paths: path/to/boost_1_61_0/stage/lib, 可以将此目录下所有动态链接库文件(如*.dylib、*.dll)拷贝到/usr/local/lib下,方便编译后的程序直接执行;

# cmake
```
cmake_minimum_required(VERSION 3.5)
project(learning_boost)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")

set(Boost_INCLUDE_DIR "/Users/jhk/Project/C++/boost_1_61_0")
set(Boost_LIBRARY_DIR "/Users/jhk/Project/C++/boost_1_61_0/stage/lib")
set(Boost_USE_STATIC_LIBS OFF)
set(Boost_USE_STATIC_RUNTIME OFF)
set(Boost_USE_MULTITHREADED ON)

find_package(Boost 1.61.0 COMPONENTS date_time)

if(Boost_FOUND)
    include_directories(${Boost_INCLUDE_DIR})
    set(SOURCE_FILES main.cpp)
    add_executable(learning_boost ${SOURCE_FILES})
    target_link_libraries( learning_boost ${Boost_LIBRARIES} )

endif()
```

# timer
```c++
#include <iostream>
#include <boost/timer.hpp>
using namespace std;
using namespace boost;

int main() {
    timer t;
    cout << t.elapsed() << endl;
    return 0;
}
```
* `elapsed()`返回timer变量从构造到调用的时间间隔，单位秒

# progress_timer
```c++
#include <iostream>
#include <boost/timer.hpp>
using namespace std;
using namespace boost;

int main() {
    progress_timer t;
    return 0;
}
```
* progress_timer对象析构时自动打印从对象创建到析构的时间间隔

# date
### 基本使用
```c++
#include <iostream>
#include <boost/date_time/gregorian/gregorian.hpp>
using namespace std;
using namespace boost::gregorian;

int main() {
    date d1(2016, 1, 1);
    date d2 = from_string("2016-02-01");
    date d3 = from_undelimited_string("20160301");
    date d4(day_clock::universal_day());

    cout << to_iso_extended_string(d1) << endl;     // 2016-01-01
    cout << to_iso_string(d2) << endl;              // 20160201
    cout << d3.week_number() << endl;               // 9
    cout << (d4 == date(not_a_date_time)) << endl;  // 0

    return 0;
}
```
特殊值:
* pos_infin         正无限
* neg_infin         负无限
* not_a_date_time   无效时间
* min_date_time     最小日期
* max_date_time     最大日期

### date_duration/days/weeks/months/years

```c++
date d(2016, 1, 1);
d += days(1);   // 2016-01-02
d += weeks(1);  // 2016-01-09
d += months(1); // 2016-02-09
d += years(1);  // 2017-02-09
```

### date_iterator/day_iterator/week_iterator/month_iterator/year_iterator
```c++
date d(2016, 1, 1);
day_iterator iterator(d);
cout << *(++iterator) << endl;  // 2016-01-02
```

# time
### time_duration/hours/mimutes/seconds/milliseconds
```c++
#include <iostream>
#include <boost/date_time/posix_time/posix_time.hpp>
using namespace std;
using namespace boost::posix_time;

int main() {

    time_duration d1(1, 10, 30);                            // 01:10:30
    time_duration d2(1, 10, 30, 1000 * 1000 * 5 + 1000);    // 01:10:35.001000

    hours h(1);
    minutes m(10);
    seconds s(30);
    millisec mis(1);
    microsec mos(1);

    time_duration d3 = h + m + s + mis + mos;               // 01:10:30.001001
    time_duration d4 = duration_from_string("-01:10:30.001001");

    string str = to_simple_string(d4);      // -01:10:30.001001
    bool b = d4.is_negative();              // 1
    time_duration d5 = d4.invert_sign();    // 01:10:30.001001

    time_duration d6(not_a_date_time);

    return 0;
}
```

### ptime
```c++
#include <iostream>
#include <boost/date_time/posix_time/posix_time.hpp>
using namespace std;
using namespace boost::posix_time;
using namespace boost::gregorian;

int main() {

    ptime p1(date(2016, 1, 1), hours(1));       // 2016-01-01 01:00:00
    ptime p2 = time_from_string("2016-01-01 01:00:00");
    ptime p3 = second_clock::local_time();      // 2016-08-13 11:17:57  东八区
    ptime p4 = second_clock::universal_time();  // 2016-08-13 03:17:57
    ptime p5(not_a_date_time);

    date d = p1.date();                         // 2016-01-01
    time_duration t = p1.time_of_day();         // 01:00:00
    string s = to_iso_extended_string(p2);      // 2016-01-01T01:00:00
    bool b = p5.is_special();                   // 1

    return 0;
}
```
* ptime 相当于 date + time_duration

# scoped_ptr
```c++
#include <iostream>
#include <boost/smart_ptr.hpp>
using namespace std;
using namespace boost;

int main() {

    scoped_ptr<string> p1(new string("TEST"));
    string str = *p1;

    scoped_array<int> p2(new int[10]);
    p2[9] = 9;

    int *q = p2.get();

//    delete q;

    return 0;
}
```
* scoped_ptr禁止拷贝构造和赋值操作,保证了它所管理的指针不被转让所有权;
* get()返回原始指针,不能对这个指针进行delete操作,否则scoped_ptr对象析构时会再次进行delete,发生未定义行为;

# shared_ptr
```c++
#include <iostream>
#include <boost/smart_ptr.hpp>
using namespace std;
using namespace boost;

int main() {

    boost::shared_ptr<string> p1(new string("abc"));
    boost::shared_ptr<string> p2 = p1;

    cout << (p1 == p2) << endl;             // 1

    cout << p1.unique() << endl;            // 0
    p2.reset();
    cout << p1.unique() << endl;            // 1

    boost::shared_ptr<int> p3;

    return 0;
}
```
* shared_ptr的实现是引用类型的智能指针,可以被自由的拷贝和赋值;
* unique()在shared_ptr是指针的唯一所有者时返回true;
* 使用内部指针进行比较运算, 即 p1 == p2 相当于 p1.get() == p2.get();
* 无参的shared_ptr()创建一个持有空指针的shared_ptr;

```c++
class Parent {
private:
    int id;

public:
    int getId() const {
        return id;
    }

    void setId(int id) {
        Parent::id = id;
    }
};

class Child : public Parent {

};

int main() {

    boost::shared_ptr<Child> child(new Child());
    child->setId(1);

    boost::shared_ptr<Parent> parent = boost::dynamic_pointer_cast<Parent>(child);
    cout << parent->getId() << endl;

    return 0;
}
```
* dynamic_pointer_cast<T> 和 dynamic_cast<T> 类似,支持shared_ptr的类型转换;

# weak_ptr
```c++
#include <iostream>
#include <boost/smart_ptr.hpp>
using namespace std;
using namespace boost;

int main() {

    boost::shared_ptr<int> sharedPtr(new int(1));
    boost::weak_ptr<int> weakPtr(sharedPtr);

    if(!weakPtr.expired()) {

        cout << weakPtr.use_count() << endl;        // 1

        boost::shared_ptr<int> p = weakPtr.lock();

        cout << weakPtr.use_count() << endl;        // 2

    }

    return 0;
}
```
* weak_ptr没有共享资源,它的构造不会引起指针引用计数的增加;
* weak_ptr没有重载operator*和->, 仅观测资源的使用情况;
* use_count()获取资源的引用计数, expired() 相当于 use_count() == 0;
* lock()从观测资源中创建一个可用的shared_ptr对象;

# pool
```c++
#include <iostream>
#include <boost/pool/pool.hpp>
using namespace std;
using namespace boost;

int main() {

    pool<> pl(sizeof(int));

    for(int i = 0; i < 10; ++i) {
        int *p = (int *) pl.malloc();
        *p = i;
    }

    int *p = (int *) pl.ordered_malloc(40);
    for(int i = 0; i < 30; ++i) {
        cout << pl.is_from(p) << endl;  // 1
        p++;
    }
    pl.free(p);

    return 0;
}
```
* pool对象析构时,会自动释放其申请的内存空间,也可以显示调用free()进行释放;
* pool(int)构造函数中的参数指定一次申请的内存大小;
* ordered_malloc(int)用于连续分配大量的内存;
* is_from测试指针地址的归属
* 内存分配失败返回0

# object_pool
```c++
#include <iostream>
#include <boost/pool/object_pool.hpp>
using namespace std;
using namespace boost;

class Demo {
public:
    Demo(int a, int b) {
        this->a = a;
        this->b = b;
    }

    ~Demo() {
        cout << "destroy..." << endl;
    }

    int a;
    int b;
};

int main() {

    object_pool<Demo> pl;

    Demo *p1 = pl.malloc();
    cout << p1->a << endl;  // 未定义行为
    pl.free(p1);            // 不调用析构函数

    Demo *p2 = pl.construct(1, 2);
    cout << p2->b << endl;  // 2
    pl.destroy(p2);         // 调用析构函数后调用free

    return 0;
}
```
* malloc()直接分配内存,不调用构造函数;
* construct()会在调用类的构造函数;
* destroy()会先调用析构函数后调用free()
* 内存分配失败返回0

# single_pool
```c++
#include <iostream>
#include <boost/pool/singleton_pool.hpp>
using namespace std;
using namespace boost;

typedef singleton_pool<double, sizeof(double)> spl;

int main() {

    double * p = (double *) spl::malloc();
    spl::free(p);

    spl::release_memory();

    return 0;
}
```
* single_pool跟pool的接口完全一致,但成员函数均是静态的;
* single_pool是单件的,且生命周期跟整个程序同样长,因此不会自动释放所占内存,除非手动调用release_memory();

# noncopyable
```c++
#include <boost/utility.hpp>

class Test : public boost::noncopyable {
};
```
* 继承noncopyable的类不可复制构造和赋值;

# optional
```c++
#include <iostream>
#include <vector>
#include <boost/optional.hpp>
using namespace std;
using namespace boost;

int main() {

    optional<string> a("xxx");
    cout << (*a) << endl;                   // xxx

    optional<int> b;
    cout << b.get_value_or(1) << endl;      // 1
    cout << b.is_initialized() << endl;     // 0

    vector<int> v({1, 2, 3, 4, 5});
    optional<vector<int> > c(v);
    c->push_back(6);

    cout << c->size() << endl;              // 6
    cout << v.size() << endl;               // 5

    return 0;
}
```
* optional<T> 要求类型T具有拷贝语义, 其内部会保存值的拷贝

# singleton
```c++
#include <iostream>
#include <boost/serialization/singleton.hpp>
using namespace std;
using namespace boost::serialization;

class A {
public:
    void out() const {
        cout << "out" << endl;
    }

    void print() {
        cout << "print" << endl;
    }
};

class B : public singleton<B> {
public:
    void out() const {
        cout << "out" << endl;
    }

    void print() {
        cout << "print" << endl;
    }
};

int main() {

    typedef singleton<A> a;
    a::get_const_instance().out();      // out
    a::get_mutable_instance().print();  // print
    
    B::get_const_instance().out();      // out
    B::get_mutable_instance().print();  // print

    return 0;
}
```
* get_const_instance()返回常对象,只能访问类的

```c++
#include <iostream>
#include <boost/operators.hpp>
using namespace std;

class A : public boost::equality_comparable<A> {
public:
    int id;

    A(int id) {
        this->id = id;
    }

    friend bool operator==(A a1, A a2) {
        return a1.id == a2.id;
    }
};

class B : public boost::less_than_comparable<B> {
public:
    int id;

    B(int id) {
        this->id = id;
    }

    friend bool operator<(B b1, B b2) {
        return b1.id < b2.id;
    }
};

int main() {

    A a1(1);
    A a2(2);
    cout << (a1 == a2) << endl; // 0
    cout << (a1 != a2) << endl; // 1

    B b1(3);
    B b2(4);
    cout << (b1 < b2) << endl;  // 1
    cout << (b1 > b2) << endl;  // 0
    cout << (b1 <= b2) << endl; // 1
    cout << (b2 >= b2) << endl; // 1

    return 0;
}
```
* equality_comparable, 需要提供==, 自动实现!=
* less_than_comparable, 需要提供<, 自动实现>、>=、<=

# uuid
```c++
#include <iostream>
#include <boost/uuid/uuid.hpp>
#include <boost/uuid/uuid_generators.hpp>
#include <boost/uuid/uuid_io.hpp>
using namespace std;
using namespace boost;

int main() {

    uuids::uuid u1 = uuids::nil_uuid();
    cout << u1.size() << endl;      // 16
    cout << u1 << endl;             // 00000000-0000-0000-0000-000000000000

    uuids::string_generator stringGenerator;
    uuids::uuid u2 = stringGenerator("0123456789abcdef0123456789abcdef");
    cout << u2 << endl;             // 01234567-89ab-cdef-0123-456789abcdef

    uuids::name_generator nameGenerator(u2);
    uuids::uuid u3 = nameGenerator("test");
    cout << u3 << endl;             // 366ab4a5-4c49-5ebe-b53b-13e042946dee

    uuids::random_generator randomGenerator;
    uuids::uuid u4 = randomGenerator();
    cout << u4 << endl;             // 39fe840c-ab53-45d8-ad98-9c0fec11f1f1

    return 0;
}
```
* nil_uuid, 只生成一个全零的UUID
* string_generator, 从一个字符串中生成uuid对象, 字符串必须是UUID格式的
* name_generator, 需要先指定一个基准UUID, 然后使用字符串派生出基于这个UUID的一系列UUID
* random_generator, 采用随机数生成UUID

# lexical_cast
```c++
#include <iostream>
#include <boost/lexical_cast.hpp>
using namespace std;
using namespace boost;

int main() {
    
    try {
        int x = lexical_cast<int>("1");
        double y = lexical_cast<double>("2.2");
        string z = lexical_cast<string>(3.3);

        cout << x << endl;  // 1
        cout << y << endl;  // 2.2
        cout << z << endl;  // 3.2999999999999998
    } catch (bad_lexical_cast &e) {
        cout << e.what() << endl;
    }
    
    return 0;
}
```
* lexical_cast可以容易的在数值和字符串之间转换, 但是将浮点类型转换为字符串类型时要注意精度问题;
* lexical_cast不会对类型进行强转, 如"1.1"的字符串不能转换为int类型, 无法转换时抛出异常bad_lexical_cast

# string_algo
```c++
#include <iostream>
#include <boost/assign.hpp>
#include <boost/algorithm/string.hpp>
using namespace std;
using namespace boost;

int main() {

    string str1("aAbBcC");
    cout << to_upper_copy(str1) << endl;    // AABBCC
    cout << to_lower_copy(str1) << endl;    // aabbcc

    string str2("abcdef");
    cout << starts_with(str2, "ab") << endl;    // 1
    cout << ends_with(str2, "ef") << endl;      // 1
    cout << contains(str2, "cd") << endl;       // 1
    cout << all(str2, is_digit()) << endl;      // 0

    string str3(" 123 abc ");
    cout << trim_left_copy(str3) << endl;               // "123 abc "
    cout << trim_right_copy(str3) << endl;              // " 123 abc"
    cout << trim_copy(str3) << endl;                    // "123 abc"
    cout << trim_copy_if(str3, !is_digit()) << endl;    // "123"

    string str4("abcdabcd");
    iterator_range<string::iterator> range;
    range = find_first(str4, "ab");
    cout << (int) (range.begin() - str4.begin()) << endl;    // 0
    range = find_nth(str4, "ab", 1);
    cout << (int) (range.begin() - str4.begin()) << endl;    // 4

    string str5("abc123abc123");
    cout << replace_first_copy(str5, "abc", "ABC") << endl;     // ABC123abc123
    cout << replace_last_copy(str5, "abc", "ABC") << endl;      // abc123ABC123
    cout << replace_all_copy(str5, "abc", "ABC") << endl;       // ABC123ABC123
    cout << replace_nth_copy(str5, "abc", 1, "ABC") << endl;    // abc123ABC123
    cout << erase_first_copy(str5, "abc") << endl;              // 123abc123
    cout << erase_last_copy(str5, "abc") << endl;               // abc123123
    cout << erase_all_copy(str5, "abc") << endl;                // 123123

    string str6("1xX2XX3xX");
    deque<string> d;
    ifind_all(d, str6, "xx");
    for(auto it = d.begin(); it != d.end(); ++it) {
        cout << *it << endl;        // xX XX xX
    }

    string str7("1_2/3");
    list<iterator_range<string::iterator> > l;
    split(l, str7, is_any_of("_/"));
    for(auto it = l.begin(); it != l.end(); ++it) {
        cout << *it << endl;        // 1 2 3
    }

    vector<string> v = assign::list_of("1")("2")("3");
    cout << join(v, "_") << endl;   // 1_2_3

    return 0;
}
```
* 前缀i: 算法是大小写不敏感
* 后缀_copy: 返回处理结果的拷贝

# xpressive
```c++
template<typename BidiIter>
struct basic_regex {
    // ...
}
typedef basic_regex<std::string::const_iterator> sregex;
typedef basic_regex<char const *> cregex;

template<typename BidiIter>
struct match_results {
    // ...
}
typedef match_results<std::string::const_iterator> smatch;
typedef match_results<char const *> cmatch;
```
* basic_regex是正则表达式的基本类,它的两个typedef: sregex 和 cregex 用于操作string和C风格字符串
* match_results是保存匹配结果的基础类, 它的两个typedef: smatch 和 cmatch 用于string和C风格字符串

```c++
#include <iostream>
#include <boost/xpressive/xpressive_dynamic.hpp>
using namespace std;
using namespace boost::xpressive;

int main() {
    // regex_match
    sregex reg1 = sregex::compile(string("^(\\d{3})(\\w).*"));
    cout << regex_match(string("123abc"), reg1) << endl; // 1

    smatch smatch1;
    if(regex_match(string("123abc"), smatch1, reg1)) {
        for(auto it = smatch1.begin(); it != smatch1.end(); ++it) {
            cout << *it << endl;                           // 123abc 123 a
        }
    }

    // regex_search
    cregex reg2 = cregex::compile("(\\d)(\\w)");
    cout << regex_search("aa2bcc", reg2) << endl;   // 1

    cmatch cmatch2;
    if(regex_search("aa2bcc", cmatch2, reg2)) {
        for(int i = 0; i < cmatch2.size(); ++i) {
            cout << cmatch2[i] << endl;             // 2b 2 b
        }
    }

    // regex_replace
    cregex reg3 = cregex::compile("\\d");
    cout << regex_replace("1a2b3c", reg3, "") << endl;  // abc

    // sregex_iterator / cregex_iterator
    string str4("1a2b3c");
    sregex reg4 = sregex::compile("(\\d)(\\w)");
    sregex_iterator cur(str4.begin(), str4.end(), reg4);
    sregex_iterator end;
    while (cur != end) {
        smatch match = *cur;
        cout << match[1] << endl;       // 1 2 3
        ++cur;
    }

    // sregex_token_iterator / cregex_token_iterator
    char *str5 = (char *) "aa||bb||cc";
    cregex_token_iterator token1(str5, str5 + strlen(str5), cregex::compile("\\w+"));
    while (token1 != cregex_token_iterator()) {
        cout << *token1 << endl;        // aa bb cc
        ++token1;
    }
    cregex_token_iterator token2(str5, str5 + strlen(str5), cregex::compile("\\|\\|"), -1);
    while (token2 != cregex_token_iterator()) {
        cout << *token2 << endl;        // aa bb cc
        ++token2;
    }

    return 0;
}
```
* regex_match : 全匹配是才返回true
* regex_search : 返回第一个匹配的子串
* regex_replace : 替换所有匹配的子串
* sregex_iterator / cregex_iterator : 查找并返回所有匹配的子串
* sregex_token_iterator / cregex_token_iterator : 分词

# bind

# function

# thread

# signal2

# bio

# config









