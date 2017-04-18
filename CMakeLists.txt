cmake_minimum_required(VERSION 2.8)

if(NOT DEFINED CMAKE_TOOLCHAIN_FILE)
  if(DEFINED ENV{VITASDK})
    set(CMAKE_TOOLCHAIN_FILE "$ENV{VITASDK}/share/vita.toolchain.cmake" CACHE PATH "toolchain file")
  else()
    message(FATAL_ERROR "Please define VITASDK to point to your SDK path!")
  endif()
endif()


set(SHORT_NAME vita-restart)
project(${SHORT_NAME})
include("${VITASDK}/share/vita.cmake" REQUIRED)

set(VITA_APP_NAME "Restart")
set(VITA_TITLEID  "VRST00001")
set(VITA_VERSION "00.10")


add_executable(${SHORT_NAME}
  main.c
)


target_link_libraries(${SHORT_NAME}
  ScePower_stub
)

vita_create_self(${SHORT_NAME}.self ${SHORT_NAME} UNCOMPRESSED)
vita_create_vpk(${SHORT_NAME}.vpk ${VITA_TITLEID} ${SHORT_NAME}.self
  VERSION ${VITA_VERSION}
  NAME ${VITA_APP_NAME}
  FILE sce_sys/icon0.png sce_sys/icon0.png
)

add_custom_target(send
  COMMAND curl -T ${SHORT_NAME}.self ftp://${PSVITAIP}:1337/ux0:/app/${VITA_TITLEID}/eboot.bin
  DEPENDS ${SHORT_NAME}.self
)

add_custom_target(shellsend
  COMMAND psp2shell_cli ${PSVITAIP} 3333 reload ${SHORT_NAME}.self
  DEPENDS ${SHORT_NAME}.self
)

add_custom_target(vpksend
  COMMAND curl -T ${SHORT_NAME}.vpk ftp://${PSVITAIP}:1337/ux0:/
  DEPENDS ${SHORT_NAME}.vpk
)
