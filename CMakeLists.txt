cmake_minimum_required(VERSION 3.5)
project(serial)

# Encontrar paquetes de ROS 2
find_package(ament_cmake REQUIRED)

if(APPLE)
    find_library(IOKIT_LIBRARY IOKit)
    find_library(FOUNDATION_LIBRARY Foundation)
endif()

if(UNIX AND NOT APPLE)
    # Si es Linux, agregar rt y pthread
    set(rt_LIBRARIES rt)
    set(pthread_LIBRARIES pthread)
endif()

## Definir fuentes
set(serial_SRCS
    src/serial.cc
    include/serial/serial.h
    include/serial/v8stdint.h
)

if(APPLE)
    list(APPEND serial_SRCS src/impl/unix.cc)
    list(APPEND serial_SRCS src/impl/list_ports/list_ports_osx.cc)
elseif(UNIX)
    list(APPEND serial_SRCS src/impl/unix.cc)
    list(APPEND serial_SRCS src/impl/list_ports/list_ports_linux.cc)
else()
    list(APPEND serial_SRCS src/impl/win.cc)
    list(APPEND serial_SRCS src/impl/list_ports/list_ports_win.cc)
endif()

## Crear la biblioteca
add_library(${PROJECT_NAME} ${serial_SRCS})
target_compile_options(${PROJECT_NAME} PRIVATE -fPIC)

if(APPLE)
    target_link_libraries(${PROJECT_NAME} ${FOUNDATION_LIBRARY} ${IOKIT_LIBRARY})
elseif(UNIX)
    target_link_libraries(${PROJECT_NAME} rt pthread)
else()
    target_link_libraries(${PROJECT_NAME} setupapi)
endif()

## Incluir encabezados
include_directories(include)

## Instalar la biblioteca
install(TARGETS ${PROJECT_NAME}
    ARCHIVE DESTINATION lib
    LIBRARY DESTINATION lib
    RUNTIME DESTINATION bin
)

## Instalar headers
install(DIRECTORY include/ DESTINATION include)

## Agregar soporte ament
ament_export_include_directories(include)
ament_export_libraries(${PROJECT_NAME})

ament_package()
