# get a list of all headers
file( GLOB_RECURSE  _header_files  RELATIVE  ${CMAKE_CURRENT_LIST_DIR}  "${PROJECT_SOURCE_DIR}/include/*.h" )
# add the library target
add_library( date
    tz.cpp
    ${_header_files}  # for IDEs
)
unset( _header_files  CACHE )

set( _using_conan  FALSE )
if( TARGET  CONAN_PKG::curl )
    set( _using_conan  TRUE )
endif()
target_include_directories( date
    PUBLIC   "$<INSTALL_INTERFACE:include>"
    PUBLIC   "$<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>"
    PRIVATE  "$<$<AND:$<BOOL:${CURL_FOUND}>,$<NOT:$<BOOL:${_using_conan}>>>:${CURL_INCLUDE_DIRS}>"
)

# attempt to find a threading library
set( CMAKE_THREAD_PREFER_PTHREAD  TRUE )
set( THREADS_PREFER_PTHREAD_FLAG  TRUE )
find_package( Threads )

target_link_libraries( date
    PRIVATE  "$<$<BOOL:TARGET Threads>:Threads\:\:Threads>"
)

if( TARGET  CONAN_PKG::curl )
  target_link_libraries( date
      PRIVATE  CONAN_PKG::curl
  )
else()
  target_link_libraries( date
      PRIVATE  "$<$<BOOL:${CURL_FOUND}>:${CURL_LIBRARIES}>"
  )
endif()

target_compile_definitions( date
    PRIVATE  "$<$<BOOL:DEFINED  DATE_INSTALL>:INSTALL=${DATE_INSTALL}>"
    PRIVATE  "$<$<BOOL:DEFINED  DATE_HAS_REMOTE_API>:HAS_REMOTE_API=$<BOOL:${DATE_HAS_REMOTE_API}>>"
    PRIVATE  "$<$<BOOL:DEFINED  DATE_AUTO_DOWNLOAD>:AUTO_DOWNLOAD=$<BOOL:${DATE_AUTO_DOWNLOAD}>>"
    PRIVATE  "$<$<BOOL:DEFINED  DATE_USE_OS_TZDB>:USE_OS_TZDB=$<BOOL:${DATE_USE_OS_TZDB}>>"
    PRIVATE  "$<$<BOOL:DEFINED  DATE_USE_SHELL_API>:USE_SHELL_API=$<BOOL:${DATE_USE_SHELL_API}>>"
    PRIVATE  "$<$<BOOL:DEFINED  DATE_ONLY_C_LOCALE>:ONLY_C_LOCALE=$<BOOL:${DATE_ONLY_C_LOCALE}>>"
)
