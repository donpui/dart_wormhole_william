set(WORMHOLE_WILLIAM_HEADER ${CMAKE_CURRENT_BINARY_DIR}/libwormhole_william.h)
set(WORMHOLE_WILLIAM_INCLUDE_DIR ${CMAKE_CURRENT_BINARY_DIR})
set(WORMHOLE_WILLIAM_C_DIR ${WORMHOLE_WILLIAM_GO_DIR}/c/)


if (${CMAKE_SYSTEM_NAME} STREQUAL "Windows")
  set(WORMHOLE_WILLIAM_LIB ${CMAKE_CURRENT_BINARY_DIR}/wormhole_william.dll)
  file(TO_NATIVE_PATH ${CMAKE_CURRENT_BINARY_DIR}/wormhole_william.h OLD_HEADER_PATH)
  file(TO_NATIVE_PATH ${WORMHOLE_WILLIAM_HEADER} NEW_HEADER_PATH)
  set(CREATE_HEADER_CMD move /y ${OLD_HEADER_PATH} ${NEW_HEADER_PATH})
  set(BUILDMODE c-shared)
elseif (${CMAKE_SYSTEM_NAME} STREQUAL "iOS")
  set(WORMHOLE_WILLIAM_LIB ${CMAKE_CURRENT_BINARY_DIR}/libwormhole_william.a)
  set(CREATE_HEADER_CMD echo "${WORMHOLE_WILLIAM_HEADER} produced by go build")
  set(CC_FOR_TARGET "/usr/local/go/misc/ios/clangwrap.sh")
  set(BUILDMODE c-archive)
elseif (${CMAKE_SYSTEM_NAME} STREQUAL "Darwin")
  set(WORMHOLE_WILLIAM_LIB ${CMAKE_CURRENT_BINARY_DIR}/libwormhole_william.dylib)
  set(CREATE_HEADER_CMD echo "${WORMHOLE_WILLIAM_HEADER} produced by go build")
  set(BUILDMODE c-shared)
else()
  set(WORMHOLE_WILLIAM_LIB ${CMAKE_CURRENT_BINARY_DIR}/libwormhole_william.so)
  set(CREATE_HEADER_CMD echo "${WORMHOLE_WILLIAM_HEADER} produced by go build")
  set(BUILDMODE c-shared)
endif()

if(${CMAKE_HOST_SYSTEM_NAME} STREQUAL "Windows")
  set(EXPORT set)
else()
  set(EXPORT export)
endif()

file(GLOB_RECURSE WORMHOLE_WILLIAM_SRC_FILES
  ${WORMHOLE_WILLIAM_GO_DIR}/**.go
  ${WORMHOLE_WILLIAM_GO_DIR}/c/**.c
  ${WORMHOLE_WILLIAM_GO_DIR}/c/**.h)

add_custom_command(
  OUTPUT ${WORMHOLE_WILLIAM_LIB} ${WORMHOLE_WILLIAM_HEADER}
  COMMENT Building wormhole_william shared library
  COMMAND
    ${EXPORT} CGO_ENABLED=1&&
    ${EXPORT} GOARCH=${GOARCH}&&
    ${EXPORT} GOOS=${GOOS}&&
    ${EXPORT} CC_FOR_TARGET=${CC_FOR_TARGET}&&
    ${EXPORT} CC=${CC_FOR_TARGET}&&
    go build -buildmode=${BUILDMODE} -o ${WORMHOLE_WILLIAM_LIB} ${WORMHOLE_WILLIAM_C_DIR}
  COMMAND ${CREATE_HEADER_CMD}
  USES_TERMINAL
  DEPENDS ${WORMHOLE_WILLIAM_SRC_FILES}
  WORKING_DIRECTORY ${WORMHOLE_WILLIAM_GO_DIR})

add_custom_target(
  wormhole_william_target DEPENDS ${WORMHOLE_WILLIAM_LIB} ${WORMHOLE_WILLIAM_HEADER})

if (${CMAKE_SYSTEM_NAME} STREQUAL "iOS")
  add_library(
    wormhole_william
    STATIC
    IMPORTED
    GLOBAL)
else ()
  add_library(
    wormhole_william
    SHARED
    IMPORTED
    GLOBAL)
endif()


add_dependencies(wormhole_william wormhole_william_target)

set_target_properties(
  wormhole_william
  PROPERTIES LINKER_LANGUAGE CXX
             IMPORTED_LOCATION ${WORMHOLE_WILLIAM_LIB}
             PUBLIC_HEADER ${WORMHOLE_WILLIAM_HEADER}
             BUILD_WITH_INSTALL_RPATH TRUE)
