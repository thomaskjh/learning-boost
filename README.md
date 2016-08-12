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
#### 基本使用
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

    day_iterator iter(d4);
    for(int i = 0; i < 3; ++i) {
        date d = *iter;
        cout << d.day_of_week() << endl;            // Thu Fri Sat
        ++iter;
    }

    return 0;
}
```
特殊值:
* pos_infin         正无限
* neg_infin         负无限
* not_a_date_time   无效时间
* min_date_time     最小日期
* max_date_time     最大日期

#### 日期长度 days/weeks/months/years

```c++
date d(2016, 1, 1);
d += days(1);   // 2016-01-02
d += weeks(1);  // 2016-01-09
d += months(1); // 2016-02-09
d += years(1);  // 2017-02-09
```

#### 日期区间 date_period
```c++
date_period p1(date(2016, 1, 1), date(2016, 1, 3));
date_period p2(date(2016, 1, 2), days(3));
date_period p3 = p1.intersection(p2);
date_period p4 = p1.merge(p2);

cout << p2.contains(date(2016, 1, 5)) << endl;  // 0
cout << p4.begin() << "," << p4.end() << endl;  // 2016-Jan-01,2016-Jan-05
```
* date_period是一个左闭右开区间
* intersection求区间交集, merge求区间并集

#### 日期迭代器 date_iterator/day_iterator/week_iterator/month_iterator/year_iterator
```c++
date d(2016, 1, 1);
day_iterator iterator(d);
cout << *(++iterator) << endl;  // 2016-01-02
```


