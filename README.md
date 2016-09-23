# install
`boost_1_61_0.tar.gz`下载并解压，执行以下命令：
```shell
./bootstrap.sh
./b2

```
* include paths：path/to/boost_1_61_0
* library paths：path/to/boost_1_61_0/stage/lib，可以将此目录下所有动态链接库文件(如*.dylib、*.dll)拷贝到/usr/local/lib下，方便编译后的程序直接执行

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

# std::tuple
```c++
#include <iostream>
using namespace std;
int main() {

    tuple<int, double> t1(1, 2.0);
    cout << std::get<0>(t1) << endl;    // 1

    tuple<int, double, string> t2 = make_tuple(1, 2.0, "3");
    cout << std::get<2>(t2) << endl;    // 3

    return 0;
}
```

# std::unique_ptr
```c++
#include <iostream>
using namespace std;

unique_ptr<int> f() {
    return unique_ptr<int>(new int(5));
}

int main() {

    unique_ptr<string> p1(new string("TEST"));
    string *ptr = p1.get();
    cout << *ptr << endl;                   // TEST

    unique_ptr<int> p2 = f();
    cout << *p2 << endl;                    // 5

    // unique_ptr<int> p3 = p2;
    unique_ptr<int> p3 = std::move(p2);
    cout << (p2.get() == nullptr) << endl;  // 1
    cout << *p3 << endl;                    // 5

    return 0;
}
```
* unique_ptr禁止拷贝构造和赋值操作，但是可以作为函数返回值使用；
* get()返回原始指针,不能对这个指针进行delete操作,否则unique_ptr对象析构时会再次进行delete,发生未定义行为；
* 如果确实要转移指针所有权，必须使用std::move进行转移，转移后原unique_ptr将持有一个空指针；

# std::shared_ptr
```c++
#include <iostream>
using namespace std;

int main() {

    shared_ptr<string> p1(new string("abc"));
    shared_ptr<string> p2 = p1;

    cout << (p1 == p2) << endl;             // 1

    cout << p1.unique() << endl;            // 0
    p2.reset();
    cout << p1.unique() << endl;            // 1

    shared_ptr<int> p3;

    return 0;
}
```
* shared_ptr的实现是引用类型的智能指针,可以被自由的拷贝和赋值;
* unique()在shared_ptr是指针的唯一所有者时返回true;
* 使用内部指针进行比较运算, 即 p1 == p2 相当于 p1.get() == p2.get();
* 无参的shared_ptr()创建一个持有空指针的shared_ptr;

```c++
#include <iostream>
using namespace std;

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

    shared_ptr<Child> child(new Child());
    child->setId(1);

    shared_ptr<Parent> parent = dynamic_pointer_cast<Parent>(child);
    cout << parent->getId() << endl;

    return 0;
}
```
* dynamic_pointer_cast<T> 和 dynamic_cast<T> 类似,支持shared_ptr的类型转换;

# std::weak_ptr
```c++
#include <iostream>
using namespace std;

int main() {

    shared_ptr<int> sharedPtr(new int(1));
    weak_ptr<int> weakPtr(sharedPtr);

    if(!weakPtr.expired()) {
        cout << weakPtr.use_count() << endl;        // 1
        shared_ptr<int> p = weakPtr.lock();
        cout << weakPtr.use_count() << endl;        // 2
    }

    return 0;
}
```
* weak_ptr没有共享资源,它的构造不会引起指针引用计数的增加;
* weak_ptr没有重载operator*和->, 仅观测资源的使用情况;
* use_count()获取资源的引用计数, expired() 相当于 use_count() == 0;
* lock()从观测资源中创建一个可用的shared_ptr对象;


# std::unordered_map、std::unordered_set
```c++
#include <iostream>
#include <unordered_map>
#include <unordered_set>

using namespace std;
int main() {

    unordered_map<int, string> a = {{2, "a"}, {4, "b"}, {3, "c"}};
    for(auto v : a) {
        cout << v.first << ", " << v.second << endl;
    }

    unordered_set<int> b = {7, 3, 5};
    for(auto v : b) {
        cout << v << endl;
    }

    return 0;
}
```
* unordered_map、unordered_set的功能跟map、set差不多, 只是内部使用散列表代替二叉树实现, 提高访问效率, 但插入的元素是无序的
* 因为要使用散列函数, 因此键值需要提供operator==操作

# std::system
```c++
#include <iostream>
using namespace std;

class my_category : public error_category {
public:
    virtual const char* name() const _NOEXCEPT {
        return "my_category";
    }

    virtual string message(int ev) const {
        string msg;
        switch (ev) {
            case 0:
                msg = "ok";
                break;

            case 1:
                msg = "error";
                break;
            default:
                msg = "unknown error";
        }
        return msg;
    }
};

int main() {

    try {
        throw system_error(1, my_category());
    } catch (system_error &e) {
        cout << e.what() << endl;   // error
    }
    
    try {
        throw  system_error(error_code((int) errc::filename_too_long, system_category()));
    } catch (system_error &e) {
        cout << e.what() << endl;                           // File name too long
        
        const error_code &code = e.code();
        const error_category &category = code.category();
        cout << code.message() << endl;                     // File name too long
        cout << category.message(code.value()) << endl;     // File name too long
    }

    return 0;
 }
```
* 错误类别error_category，成员函数name()获取类别名称，message()获取错误代码error_code对应的描述信息；
* 自定义类别时，需要继承error_category，必须实现name()和message()，因为它们是纯虚函数；
* 错误代码error_code，无参构造函数表示无错误，错误值为0；或者使用一个整数错误值和错误类别构造；value()获取错误值，message()等价于category().message()
* 异常类system_error，继承std::runtime_error，构造是需要error_code对象；

# std::result_of
```c++
#include <iostream>

using namespace std;

template <typename V>
class Base {
public:
    V operator()(V t) {
        return t;
    }
};

template <typename T, typename V>
typename::result_of<T(V)>::type func(T t, V v) {
    return t(v);
};

int main() {

    result_of<Base<int>(int)>::type v;  // int类型

    Base<int> b;
    v = func(b, 2);                     // 2

    return 0;
}
```

# std::ref
```c++
#include <iostream>

using namespace std;

int main() {

    int a = 1;
    reference_wrapper<int> r1 = ref(a);
    r1.get() = 2;
    cout << a << endl;      // 2

    return 0;
}
```
* ref()函数返回reference_wrapper包装类型，通过get()可以被包装对象的引用；


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
* get_const_instance()返回常对象,只能访问类的const成员

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

# BOOST_ASSERT
```c++
#include <iostream>
#include <boost/assert.hpp>
using namespace std;

int main() {

    int x = 1;
    
    BOOST_ASSERT(x == 2 && "x is error");

    return 0;
}
```
* BOOST_ASSERT仅会在debug模式下生效, 在release模式下不会进行编译, 不会影响运行效率, 还可以使用 && 向表达式增加断言的描述信息



# dynamic_bitset
```c++
#include <iostream>
#include <boost/dynamic_bitset.hpp>
#include <boost/utility/binary.hpp>
using namespace std;
using namespace boost;
int main() {

    dynamic_bitset<> d1;
    d1.resize(65, true);
    cout << d1.size() << ", " << d1.num_blocks() << endl;   // 65, 2
    cout << sizeof(unsigned long) * 8 << endl;              // 64

    dynamic_bitset<unsigned int> d2(string("0110"));
    cout << d2.num_blocks() << endl;                        // 1

    dynamic_bitset<> d3(3, 2);
    cout << d3 << endl;                                     // 010

    dynamic_bitset<> d4(3, BOOST_BINARY(101));
    cout << d4 << endl;                                     // 101

    cout << d4[1] << endl;                                  // 0
    cout << d4.to_ulong() << endl;                          // 5

    cout << (d3 & d4) << endl;                              // 000
    cout << (d3 | d4) << endl;                              // 111

    return 0;
}
```
* 第一个模板参数Block指示dynamic_bitset以什么整数类型存储二进制,必须是无符号整数, 默认是 unsigned long;
* num_blocks()返回二进制位占用Block的数量;

# bimap
```c++
#include <iostream>
#include <boost/bimap.hpp>

using namespace std;
using namespace boost;

int main() {

    bimap<int, string> bm;

    bm.left.insert(make_pair(1, string("a")));
    bm.right.insert(make_pair("b", 2));

    for(auto v : bm.left) {
        cout << v.first << ", " << v.second << endl;    // 1, a   2, b
    }

    cout << bm.left.at(1) << endl;                      // a
    cout << bm.right.at("a") << endl;                   // 1

    bm.left.insert(make_pair(3, "b"));                  // 无效操作
    bimap<int, string>::left_iterator it;
    it = bm.left.find(3);
    cout << (it == bm.left.end()) << endl;              // 1

    auto left_it = bm.left.find(2);
    cout << left_it->first << ", " << left_it->second << endl;      // 2, b

    auto right_it = bm.project_right(left_it);
    cout << right_it->first << ", " << right_it->second << endl;    // b, 2

    cout << bm.left.replace_data(left_it, "a") << endl;             // 0
    cout << bm.left.replace_key(left_it, 3) << endl;                // 1
    cout << bm.left.replace(left_it, make_pair(2, "c")) << endl;    // 1

    return 0;
}
```
* bimap的key/value值对必须都是唯一的， 插入任何一个重复的值都是无效操作；
* project_left/project_right可以转换left_iterator和right_iterator;
* replace可以替换原始的键值，但如果键或值已经存在，则替换失败；

# circular_buffer
```c++
#include <iostream>
#include <boost/circular_buffer.hpp>
using namespace std;
using namespace boost;
int main() {

    circular_buffer<int> cb(5);
    cb.push_back(1);
    cb.push_back(2);
    cb.push_back(3);

    cout << cb.size() << endl;      // 3
    cout << cb.capacity() << endl;  // 5

    cb.push_front(4);
    cb.push_front(5);

    for(int v : cb) {
        cout << v << endl;  // 5 4 1 2 3
    }

    cb.push_back(6);
    for(circular_buffer<int>::iterator it = cb.begin(); it != cb.end(); ++it) {
        cout << *it << endl;    // 4 1 2 3 6
    }

    cb.rotate(cb.begin() + 2);
    for(int v : cb) {
        cout << v << endl;      // 2 3 6 4 1
    }

    return 0;
}
```
* 当元素达到容器的容量上限时，将自动重用最初的空间；
* rotate()从指定的迭代器位置旋转整个缓存区；

# any
```c++
#include <iostream>
#include <boost/any.hpp>
using namespace std;
using namespace boost;
int main() {

    int i = 1;
    any a(i);
    ++i;
    cout << i << ", " << any_cast<int>(a) << endl;                      // 2 1

    a = std::shared_ptr<int>(new int);
    std::shared_ptr<int> ptr = any_cast<std::shared_ptr<int>>(a);
    cout << ptr.use_count() << endl;                                    // 2

    a = ref(i);
    ++i;
    cout << i << ", " << any_cast<reference_wrapper<int>>(a) << endl;   // 3 3

    cout << ptr.use_count() << endl;                                    // 1

    return 0;
}
```
* any类不是模板类，但其构造函数是模板函数；
* any类存储的是对象的拷贝，为了避免大对象的拷贝代价，可以使用ref进行包装；
* any类不应该持有原始指针，应该使用智能指针进行包装，这样析构时就智能指针就会调用delete；

# variant
```c++
#include <iostream>
#include <boost/variant.hpp>
using namespace std;
using namespace boost;
int main() {

    variant<int, double, string> v;
    cout << v.empty() << endl;      // 1

    v = 0.1;
    cout << v.which() << endl;                      // 1
    if(v.type() == typeid(int)) {                   // double
        cout << "int" << endl;
    } else if(v.type() == typeid(double)) {
        cout << "double" << endl;
    } else if(v.type() == typeid(string)) {
        cout << "string" << endl;
    }

    cout << get<double>(v) << endl;                 // 0.1

    return 0;
}
```
* variant是有界类型，允许保存的数据类型必须在模板参数列表中声明；
* empty()用来检测当前variant是否持有对象；
* which()返回variant当前值的类型在模板参数列表的索引号（从0开始计数）；
* type()检测variant的类型；

# multi_array
```c++
#include <iostream>
#include <boost/multi_array.hpp>
using namespace std;
using namespace boost;
int main() {

    multi_array<int, 3> ma(extents[1][2][3]);

    unsigned long dimensions = ma.num_dimensions();     // 3
    const unsigned long *shape = ma.shape();
    for(int i = 0; i < dimensions; ++i) {
        cout << shape[i] << endl;                       // 1 2 3
    }

    for(int i = 0;i < 1; ++i) {
        for(int j = 0; j < 2; ++j) {
            for(int k = 0; k < 3; ++k) {
                ma[i][j][k] = i * 2 * 3 + j * 3 + k;
            }
        }
    }

    const int *p = ma.data();
    for(int i = 0; i < ma.num_elements(); ++i) {
        cout << *(p + i) << endl;                       // 0 1 2 3 4 5
    }

    boost::array<std::size_t, 3> s = {3, 2, 1};
    ma.reshape(s);
    cout << ma.shape()[0] << endl;                      // 3
    cout << ma[2][1][0] << endl;                        // 5

    s = {2, 2, 2};
    ma.resize(s);
    for(int i = 0; i < 2; ++i) {
        for(int j = 0; j < 2; ++j) {
            for(int k = 0; k < 2; ++k) {
                cout << ma[i][j][k] << endl;            // 0 0 1 0 2 0 3 0
            }
        }
    }
    
    return 0;
}
```
* reshape()可以变动各个维的大小，但是变动前后维度成绩必须相同，也就是元素数量保持不变；
* resize()将重新分配内存，原有的相同坐标下的元素将复制到新内存，例如(0,0,0)坐标的元素将被复制到(0,0,0)，新增的元素将使用缺省构造函数构造；

# 
```xml
<?xml version="1.0" encoding="utf-8"?>
<conf>
    <int>1</int>
    <bool>false</bool>
    <string>test</string>
    <vector>
        <v>1</v>
        <v>2</v>
        <v>3</v>
    </vector>
</conf>
```
```c++
#include <iostream>
#include <boost/property_tree/ptree.hpp>
#include <boost/property_tree/xml_parser.hpp>
using namespace std;
using namespace boost::property_tree;
int main() {

    ptree pt;
    read_xml("/Users/jhk/Project/C++/learning-boost/conf.xml", pt);

    cout << pt.get<int>("conf.int") << endl;                    // 1
    cout << pt.get<bool>("conf.bool") << endl;                  // 0
    cout << pt.get<string>("conf.string") << endl;              // test
    cout << pt.get<double>("conf.default", 2.0) << endl;        // 2.0

    pt.put("conf.vector.v", "4");
    auto p = pt.get_child("conf.vector");
    for(auto it = p.begin(); it != p.end(); ++it) {
        cout << it->second.get_value<int>() << endl;            // 4 2 3
    }

    pt.add("conf.vector.v", "5");
    p = pt.get_child("conf.vector");
    for(auto it = p.begin(); it != p.end(); ++it) {
        cout << it->second.get_value<int>() << endl;            // 4 2 3 5
    }

    write_xml("/Users/jhk/Project/C++/learning-boost/conf.xml", pt);
    return 0;
 }
```
* put()当原有节点存在时修改元素，否则才新建节点；
* add()新建节点；

```
;comment
conf {
    int 1
    bool false
    string test
    vector {
        v 1
        v 2
        v 3
    }
}
```
```c++
#include <iostream>
#include <boost/property_tree/ptree.hpp>
#include <boost/property_tree/info_parser.hpp>
using namespace std;
using namespace boost::property_tree;
int main() {

    ptree pt;
    read_info("/Users/jhk/Project/C++/learning-boost/conf.info", pt);

    cout << pt.get<int>("conf.int") << endl;                    // 1
    // ...
    return 0;
 }
```

# filesystem
### 基本操作
```c++
#include <iostream>
#include <boost/filesystem.hpp>

using namespace std;
using namespace boost::filesystem;

int main() {

    path p("./a/b");
    cout << p.string() << endl;         // "./a/b"
    cout << p.is_absolute() << endl;    // 0

    p /= "c.txt";
    cout << p.string() << endl;         // "./a/b/c.txt"

    cout << p.parent_path() << endl;    // "./a/b"
    cout << p.filename() << endl;       // "c.txt"
    cout << p.stem() << endl;           // "c"
    cout << p.extension() << endl;      // ".txt"

    cout << portable_file_name("file.txt") << endl;     // 1
    cout << portable_file_name("./file.txt") << endl;   // 0
    cout << portable_directory_name("dir") << endl;     // 1
    cout << portable_directory_name("/dir") << endl;    // 0
    cout << portable_directory_name("./dir") << endl;   // 0

    return 0;
}
```
* portable_file_name：检查文件名是否符合规范，不能以点号或连字符开头；
* portable_directory_name：检查目录是否符合规范，名字中不能出现点号；

### 异常处理和常用方法
```c++
#include <iostream>
#include <boost/filesystem.hpp>

using namespace std;
using namespace boost::filesystem;

int main() {

    try {
        cout << initial_path() << endl;                 // "/Users"
        cout << current_path() << endl;                 // "/Users"
        cout << file_size(path("/bin/sh")) << endl;     // 632672

        create_directory("/Users/jhk/a");
        create_directories("/Users/jhk/b/c");

        if (exists("/Users/jhk/a")) {
            remove_all("/Users/jhk/a");
        }

        copy_file("/bin/sh", "/Users/jhk/a/file");

    } catch (filesystem_error &e) {
        cout << e.path1().string() << endl;
        cout << e.path2().string() << endl;
        cout << e.what() << endl;
    }

    return 0;
}
```
* initial_path()获取程序启动时的路径
* current_path()获取当前路径
* path1()和path2()获取发生异常时的path对象;
* create_directory()创建单一目录，create_directories()迭代创建目录
* remove()删除一个文件或空文件夹，remove_all()迭代删除所有；
* copy_file()拷贝文件；

### 文件状态
```c++
#include <iostream>
#include <boost/filesystem.hpp>

using namespace std;
using namespace boost::filesystem;

int main() {

    BOOST_ASSERT(status("/a/b").type() == file_not_found);
    BOOST_ASSERT(status("/bin/sh").type() == regular_file);

    cout << exists("/a/b") << endl;                 // 0
    cout << is_regular_file("/bin/sh") << endl;     // 1
    cout << is_directory("/bin/sh") << endl;        // 0

    return 0;
}
```
* file_not_found：文件不存在
* regular_file：普通文件
* directory_file：目录
* symlink_file：链接文件

### 路径迭代
```c++
#include <iostream>
#include <boost/filesystem.hpp>

using namespace std;
using namespace boost::filesystem;

int main() {

    path p("/Users/jhk/Downloads");
    recursive_directory_iterator end;
    for(recursive_directory_iterator pos(p); pos != end; ++pos) {

        path current = *pos;

        if(is_directory(current) && pos.level() > 0) {
            pos.no_push();
        }

        cout << current.string() << endl;
    }

    return 0;
}
```
* recursive_directory_iterator可以深度搜索目录，迭代当前目录及子目录下的所有文件，迭代方式类似二叉树的深度遍历；
* level()返回当前路径深度，从0开始；
* no_push()可以使目录不参与遍历；

# program_options
### 命令行参数
```c++
#include <iostream>
#include <boost/program_options.hpp>

using namespace std;
using namespace boost::program_options;

// 调用参数：./learning_boost --int=1 --vector=1.1 -v2.2 --vector=3.3
int main(int argc, char * argv[]) {

    typed_value<vector<double>> *v = value<vector<double>>();

    options_description opts("demo options");
    opts.add_options()
            ("help,h", "just a help info")
            ("int,i", value<int>(), "int argument")
            ("string,s", value<string>(q)->default_value("xx")->implicit_value("yy"), "string argument")
            ("vector,v", v->required()->multitoken(), "vector argument");
    variables_map vm;
    store(parse_command_line(argc, argv, opts), vm);

    if(vm.size() == 0 || vm.count("help")) {
        cout << opts << endl;
        return 0;
    }
    
    notify(vm);

    if(vm.count("int")) {
        cout << vm["int"].as<int>() << endl;        // 1
    }

    if(vm.count("string")) {
        cout << vm["string"].as<string>() << endl;  // xx
    }

    if(vm.count("vector")) {
        vector<double> d = vm["vector"].as<vector<double>>();
        for(double dd : d) {
            cout << dd << endl;                     // 1.1 2.2 3.3
        }
    }

    return 0;
}
```
* add_options()重载了括号运算符，用来添加选项描述器；
* 选项值的类型为typed_value，可使用value工厂函数获取；
* defalut_value()，当选项没有出现在命令行中时使用该值；
* implicit_value()，当选项出现但没有给具体值时使用该值；
* multitoken()，表示可以可以接受多个相同的选项；
* "int,i"表示同时制定长名和短名，即接受下面两种格式：--int=2 或 -i2
* stroe()和parse_command_line()用来解析命令行参数串，结果保存在variables_map，variables_map的value_type是boost::any，通过as<TYPE>)()转型获取具体值；

### 配置文件
```
# 配置文件：config
int=1
string=a
```

```c++
#include <iostream>
#include <boost/program_options.hpp>
#include <fstream>

using namespace std;
using namespace boost::program_options;

int main(int argc, char * argv[]) {

    options_description opts("file options");
    opts.add_options()
            ("help,h", "just a help info")
            ("int,i", value<int>(), "int argument")
            ("string,s", value<string>(), "string argument");

    variables_map vm;
    store(parse_config_file<char>("/Users/jhk/Project/C++/learning-boost/config", opts, true), vm);

    // 或者
    // ifstream ifs("/Users/jhk/Project/C++/learning-boost/config");
    // store(parse_config_file(ifs, opts, true), vm);

    cout << vm["int"].as<int>() << endl;            // 1
    cout << vm["string"].as<string>() << endl;      // a

    return 0;
}
```

# bind
```c++
#include <iostream>
#include <boost/bind.hpp>

using namespace std;
using namespace boost;

void f1(int a, int b) {
    cout << a << ", " << b << endl;
}

class demo {
public:
    void f2(int a, int b) {
        cout << a << ", " << b << endl;
    }
};

class func1 {
public:
    int operator()(int a, int b) {
        cout << a << ", " << b << endl;
        return 0;
    }
};

class func2 {
public:
    typedef int result_type;
    int operator()(int a, int b) {
        cout << a << ", " << b << endl;
        return 0;
    }
};

int main(int argc, char * argv[]) {

    auto b1 = bind(f1, _2, _1);
    b1(1, 2);                       // 2, 1

    auto b2 = bind(f1, _1, 4);
    b2(3);                          // 3, 4

    demo d;
    auto b3 = bind(&demo::f2, _1, 5, 6);
    b3(d);                          // 5, 6
    b3(boost::ref(d));              // 5, 6
    b3(&d);                         // 5, 6

    func1 c1;
    auto b4 = bind<int>(boost::ref(c1), 7, 8);
    b4();                           // 7, 8

    func2 c2;
    auto b5 = bind(c2, 9, 10);
    b5();                           // 9, 10

    return 0;
}
```
* _1、_2、_3、..._9是bind的参数占位符，表示函数调用的实际参数列表；
* 绑定成员函数时，成员函数前必须加上取地址操作符&，且第一个参数必须是对象或对象指针；
* 绑定函数对象时，如果函数对象没有定义result_type，则bind需要使用模板参数指明返回类型；
* bind采用拷贝的方式存储绑定对象和参数，乳沟拷贝代价很大，可以使用ref进行包装；

# function

# thread

# signal2

# bio

# config









