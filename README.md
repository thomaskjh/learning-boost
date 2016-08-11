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
```c++
#include <iostream>
#include <boost/date_time/gregorian/gregorian.hpp>
using namespace std;
using namespace boost::gregorian;

int main() {
    date d1(2016, 1, 1);
    date d2 = from_string("2016-01-11");
    date d3(day_clock::universal_day());
    days d = d2 - d1;
    cout << to_iso_extended_string(d1) << endl; // 2016-01-01
    cout << d2.is_not_a_date() << endl;         // 0
    cout << d.days() << endl;                   // 10
    cout << d1 + months(2) << endl;             // 2016-Mar-01
    day_iterator iter(d3);
    for(int i = 0; i < 10; ++i) {
        date t = *iter;
        cout << t << endl;
        ++iter;
    }

    date d4;
    cout << (d4 == date(not_a_date_time)) << endl;  // 1

    return 0;
}
```
特殊值:
* pos_infin         正无限
* neg_infin         负无限
* not_a_date_time   无效时间
* min_date_time     最小日期
* max_date_time     最大日期


