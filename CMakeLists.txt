#-------------------------------------------------------
cmake_minimum_required(VERSION 2.6)
cmake_policy(SET CMP0003 NEW)
project(NewGameSvr)
#set(PLUTO_NEW_MAKE 1)

# Cmake module setting
set(CMAKE_MODULE_PATH  ${CMAKE_MODULE_PATH}  ${CMAKE_SOURCE_DIR}/Build/Cmake)
include(${CMAKE_SOURCE_DIR}/Build/Cmake/EchoConfig.cmake)
include(${CMAKE_SOURCE_DIR}/Build/Cmake/GetDir.cmake)
find_package(Platform REQUIRED)

# Force out-of-source build
string(COMPARE EQUAL "${CMAKE_SOURCE_DIR}" "${CMAKE_BINARY_DIR}" BUILDING_IN_SOURCE)
if(BUILDING_IN_SOURCE)
    cecho(cred "This project requires an out of source build. \nRemove the file 'CMakeCache.txt' found in this directory before continuing. \nCreate a separate build directory and run 'cmake <srcs> [options]' from there.")
    message(FATAL_ERROR "exit")
endif()

# Platform check
if(PLATFORM MATCHES X86)
	cecho(cred "-- Check: ARCH ${PLATFORM} detected.")		
	#message(FATAL_ERROR "only x86_64 platform support")
else()
    cecho(cred "-- Check: ARCH ${PLATFORM} detected.")
endif()

## 依赖库定义
#####################################################################
# 系统库
set (SYSTEM_LAS z dl pthread)

# 第三方组件库
#set third party lib dir
set (3RD_PARTY_DIR ${PROJECT_SOURCE_DIR}/001_ServerLib/3RdParty)

#protobuf
set (PROTOBUF_INC ${3RD_PARTY_DIR}/protobuf-2.5.0)
link_directories(${3RD_PARTY_DIR}/protobuf-2.5.0)
set (PROTOBUF_LIB protobuf)

#zero mq
set (ZMQLIB_INC ${3RD_PARTY_DIR}/zeromq-3.2.3)
link_directories(${3RD_PARTY_DIR}/zeromq-3.2.3)
set (ZMQLIB_LIB zmq rt)

#pugi xml
include_directories (${3RD_PARTY_DIR}/pugixml-1.2)
link_directories (${3RD_PARTY_DIR}/pugixml-1.2)
set (PUGIXML_LIB pugixml)

#lua lib
set (LUALIB_INC ${3RD_PARTY_DIR}/lua-5.2.2)
link_directories(${3RD_PARTY_DIR}/lua-5.2.2)
set (LUALIB_LIB lua)

#gtest
set(GTESTLIB_INC ${3RD_PARTY_DIR}/gtest-1.6.0)
link_directories(${3RD_PARTY_DIR}/gtest-1.6.0)
set(GTESTLIB_LIB gtest)

#mysql client
set(MYSQLCLIENT_LIB mysqlclient)
link_directories(/usr/lib64/mysql/)
include_directories(/usr/include/)

#crypto++
#set(CRYPTOPP_INC ${3RD_PARTY_DIR}/cryptopp)
#set(CRYPTOPP_LIB cryptopp)
#link_directories(${3RD_PARTY_DIR}/cryptopp)

#Botan
set(BOTAN_INC ${3RD_PARTY_DIR}/botan-1.10.5)
set(BOTAN_LIB botan-1.10)
link_directories(${3RD_PARTY_DIR}/botan-1.10.5)

#libev
set(LIBEV_INC ${3RD_PARTY_DIR}/libev-4.15)
set(LIBEV_LIB ev)
link_directories(${3RD_PARTY_DIR}/libev-4.15)

#OSS UDP Log
set(OSSUDPLOG_INC ${3RD_PARTY_DIR}/KingnetUdpLog)
set(OSSUDPLOG_LIB KingnetUdpLogLib)
link_directories(${3RD_PARTY_DIR}/KingnetUdpLog)

# 001_ServerLib 
set (SERVERLIB_DIR ${PROJECT_SOURCE_DIR}/001_ServerLib)
set (SERVERLIB_INC ${SERVERLIB_DIR}/BaseLibs)
set (SERVERLIB_LAS ServerLib)
set (SERVERLIB_MTLAS ServerLibMT)
link_directories(${SERVERLIB_DIR}/BaseLibs)

# 004_Errno
set (ERROR_INC ${PROJECT_SOURCE_DIR}/004_Errno)

# 102_StarConfigEngine
set (CONFIG_DIR ${PROJECT_SOURCE_DIR}/102_ConfigEngine)
set (CONFIG_INC ${CONFIG_DIR})
set(CONFIG_LAS ConfigEngine)

# 103_Protocol
set (PROTO_DIR ${PROJECT_SOURCE_DIR}/103_Protocol/Src)

#103_Protocol config
include_directories(${PROTO_DIR}/config/)
set(TDRCONFIG_LIB tdrconfig)

#103_Protocol proto
include_directories(${PROTO_DIR}/proto/)
set (PROTOCOL_LIB protocol)

# All static lib
set (ALL_LAS ${SERVERLIB_LAS} ${CONFIG_LAS} ${SERVERLIB_LAS} ${OSSUDPLOG_LIB}
	${TDRCONFIG_LIB} ${PROTOCOL_LIB} ${LUALIB_LIB} ${ZMQLIB_LIB}
	${PROTOBUF_LIB} ${SYSTEM_LAS})
	#jasonxiong todo 后续重构修改时再添加
	#${CONFIG_LAS} ${TSF4G_LAS} {SYSTEM_LAS} ${LUALIB_LAS})
#set (ALL_MTLAS ${SERVERLIB_MTLAS} ${SYSTEM_LAS} ${CONFIG_LAS} ${PROTOLAS} ${LUALIB_LIB}
	#${ZMQLIB_LIB} ${PROTOBUF_LIB})
	#jasonxiong todo 后续重构时修改再添加
	#${CONFIG_LAS} ${TSF4G_LAS} ${SYSTEM_LAS})

# 编译选项
#####################################################################
if (NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Debug)
endif()

cecho(cgreen "-- BUILD: ${CMAKE_BUILD_TYPE}")
if (${CMAKE_BUILD_TYPE} STREQUAL "Debug" 
        OR ${CMAKE_BUILD_TYPE} STREQUAL "RelWithDebInfo")
    add_definitions(-D_DEBUG_)
    set(MT_NAME_TAG ".dbg")
else()
    remove_definitions(D_DEBUG_)
    set(MT_NAME_TAG ".run")
endif()

#这个找个要处理掉
add_definitions(-D_POSIX_MT_)
# 库文件的附加参数 -fPIC
set(ALL_FLAGS_IN_ONE_COMMON "-Wall -fPIC -rdynamic -D_FILE_OFFSET_BITS=64")
#-Werror -Wno-deprecated-declarations") 
set(ALL_FLAGS_IN_ONE_DEBUG "-g -ggdb -O0 ${ALL_FLAGS_IN_ONE_COMMON}")
set(ALL_FLAGS_IN_ONE_RELEASE "-O3 ${ALL_FLAGS_IN_ONE_COMMON} -DNDEBUG")
set(ALL_FLAGS_IN_ONE_RELWITHDEBINFO "-O2 -g -ggdb ${ALL_FLAGS_IN_ONE_COMMON}")
set(ALL_FLAGS_IN_ONE_MINSIZEREL "-Os ${ALL_FLAGS_IN_ONE_COMMON} -DNDEBUG")

set(CMAKE_CXX_FLAGS_DEBUG "$ENV{CXXFLAGS} ${ALL_FLAGS_IN_ONE_DEBUG}")
set(CMAKE_CXX_FLAGS_RELEASE "$ENV{CXXFLAGS} ${ALL_FLAGS_IN_ONE_RELEASE}")
set(CMAKE_CXX_FLAGS_RELWITHDEBINFO "$ENV{CXXFLAGS} ${ALL_FLAGS_IN_ONE_RELWITHDEBINFO}")
set(CMAKE_CXX_FLAGS_MINSIZEREL "$ENV{CXXFLAGS} ${ALL_FLAGS_IN_ONE_MINSIZEREL}")

set(CMAKE_C_FLAGS_DEBUG "$ENV{CFLAGS} ${ALL_FLAGS_IN_ONE_DEBUG}")
set(CMAKE_C_FLAGS_RELEASE "$ENV{CFLAGS} ${ALL_FLAGS_IN_ONE_RELEASE}")
set(CMAKE_C_FLAGS_RELWITHDEBINFO "$ENV{CFLAGS} ${ALL_FLAGS_IN_ONE_RELWITHDEBINFO}")
set(CMAKE_C_FLAGS_MINSIZEREL "$ENV{CFLAGS} ${ALL_FLAGS_IN_ONE_MINSIZEREL}")

message(STATUS "CMAKE_CXX_FLAGS_DEBUG =" ${CMAKE_CXX_FLAGS_DEBUG})
message(STATUS "CMAKE_CXX_FLAGS_RELEASE =" ${CMAKE_CXX_FLAGS_RELEASE})
message(STATUS "CMAKE_CXX_FLAGS_RELWITHDEBINFO =" ${CMAKE_CXX_FLAGS_RELWITHDEBINFO})
message(STATUS "CMAKE_CXX_FLAGS_MINSIZEREL =" ${CMAKE_CXX_FLAGS_MINSIZEREL})

#####################################################################
# 固定生成目录为
set (EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/release/bin)
set (LIBRARY_OUTPUT_PATH ${PROJECT_BINARY_DIR}/release/lib)

#INCLUDES
include_directories(${SERVERLIB_INC})
include_directories(${ERROR_INC})
include_directories(${LUALIB_INC})
include_directories(${ZMQLIB_INC})
include_directories(${PROTOBUF_INC})
include_directories(${PROTO_INC})
include_directories(${GTESTLIB_INC})
include_directories(${CONFIG_INC})
include_directories(${CRYPTOPP_INC})
include_directories(${OSSUDPLOG_INC})
include_directories(${LIBEV_INC})
include_directories(${BOTAN_INC})

#Link Libraries
link_directories(${LIBRARY_OUTPUT_PATH})

# Server Name define
set (GATEWAY_SVR_EXE    GatewayServer${MT_NAME_TAG})

# sub projects
add_subdirectory(001_ServerLib)
add_subdirectory(014_GatewayServer)

