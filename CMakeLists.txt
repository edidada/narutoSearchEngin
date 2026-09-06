cmake_minimum_required(VERSION 3.10)
project(narutoSearchEngine CXX)

# 与 Makefile 保持一致：C++11，-Wall -Wno-deprecated，Debug 默认带 -g
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Debug)
endif()

add_compile_options(-Wall -Wno-deprecated)

# ---------------------------------------------------------------
# 第三方依赖说明：
#   源码（如 base/src/mytask.cc、base/src/test_search.cc）写死了
#     #include "/home/wuyijian/simhash/src/Simhasher.hpp"
#     #include "/home/wuyijian/simhash/src/CppJieba/QuerySegment.hpp"
#   因此本机/CI 上必须存在该路径（例如 git clone --recursive
#   https://github.com/yanyiwu/simhash.git 后放到该位置）。
# ---------------------------------------------------------------
set(SIMHASH_HOME "/home/wuyijian/simhash")
if(NOT EXISTS "${SIMHASH_HOME}/src")
  message(WARNING
    "未找到第三方依赖目录 ${SIMHASH_HOME}/src：\n"
    "  源码中有硬编码的绝对 include，请先执行：\n"
    "  git clone --recursive https://github.com/yanyiwu/simhash.git ${SIMHASH_HOME}")
endif()

# 编译主程序 bin/main（offline/ 是独立离线建库程序，不参与主构建）
file(GLOB BASE_SOURCES ${CMAKE_CURRENT_SOURCE_DIR}/base/src/*.cc)
file(GLOB NET_SOURCES  ${CMAKE_CURRENT_SOURCE_DIR}/net/src/*.cc)

add_executable(main ${BASE_SOURCES} ${NET_SOURCES})

target_include_directories(main PRIVATE
  ${CMAKE_CURRENT_SOURCE_DIR}/base/inc
  ${CMAKE_CURRENT_SOURCE_DIR}/net/inc
)

# 与 Makefile 的 LIBS := -llog4cpp -ljson -lhiredis -lpthread 对应
target_link_libraries(main PRIVATE
  log4cpp
  json
  hiredis
  pthread
)

# 可执行文件输出到源树的 bin/，与 Makefile/start.sh 行为一致
set_target_properties(main PROPERTIES
  RUNTIME_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/bin)
