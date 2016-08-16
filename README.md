# install
`boost_1_61_0.tar.gz`下载并解压, 执行以下命令:
```shell
./bootstrap.sh
./b2
```
* include paths: path/to/boost_1_61_0
* library paths: path/to/boost_1_61_0/stage/lib

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

