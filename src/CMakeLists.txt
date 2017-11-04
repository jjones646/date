# stringify the database files if making portable library
if ( DATE_MAKE_PORTABLE )
    function( minimize_tzdb_string  INSTRING  OUT )
        set( _working_string  ${INSTRING} )
        string( REGEX  REPLACE  "#([^\n]+[\n]+)?"  "\n"  _working_string  ${_working_string} )
        string( REGEX  REPLACE  "([^\n]+)\n"  "\"\\1\\\\n\"\n"  _working_string  ${_working_string} )
        set( ${OUT}  ${_working_string}  PARENT_SCOPE )
    endfunction()

    file( READ  "${DATE_INSTALL_DIR}/africa"  TZDB_AFRICA )
    file( READ  "${DATE_INSTALL_DIR}/antarctica"  TZDB_ANTARCTICA )
    file( READ  "${DATE_INSTALL_DIR}/asia"  TZDB_ASIA )
    file( READ  "${DATE_INSTALL_DIR}/australasia"  TZDB_AUSTRALASIA )
    file( READ  "${DATE_INSTALL_DIR}/backward"  TZDB_BACKWARD )
    file( READ  "${DATE_INSTALL_DIR}/etcetera"  TZDB_ETCETERA )
    file( READ  "${DATE_INSTALL_DIR}/europe"  TZDB_EUROPE )
    file( READ  "${DATE_INSTALL_DIR}/pacificnew"  TZDB_PACIFICNEW )
    file( READ  "${DATE_INSTALL_DIR}/northamerica"  TZDB_NORTHAMERICA )
    file( READ  "${DATE_INSTALL_DIR}/southamerica"  TZDB_SOUTHAMERICA )
    file( READ  "${DATE_INSTALL_DIR}/systemv"  TZDB_SYSTEMV )
    file( READ  "${DATE_INSTALL_DIR}/leapseconds"  TZDB_LEAPSECONDS )

    minimize_tzdb_string( "${TZDB_AFRICA}"  TZDB_AFRICA )
    minimize_tzdb_string( "${TZDB_ANTARCTICA}"  TZDB_ANTARCTICA )
    minimize_tzdb_string( "${TZDB_ASIA}"  TZDB_ASIA )
    minimize_tzdb_string( "${TZDB_AUSTRALASIA}"  TZDB_AUSTRALASIA )
    minimize_tzdb_string( "${TZDB_BACKWARD}"  TZDB_BACKWARD )
    minimize_tzdb_string( "${TZDB_ETCETERA}"  TZDB_ETCETERA )
    minimize_tzdb_string( "${TZDB_EUROPE}"  TZDB_EUROPE )
    minimize_tzdb_string( "${TZDB_PACIFICNEW}"  TZDB_PACIFICNEW )
    minimize_tzdb_string( "${TZDB_NORTHAMERICA}"  TZDB_NORTHAMERICA )
    minimize_tzdb_string( "${TZDB_SOUTHAMERICA}"  TZDB_SOUTHAMERICA )
    minimize_tzdb_string( "${TZDB_SYSTEMV}"  TZDB_SYSTEMV )
    minimize_tzdb_string( "${TZDB_LEAPSECONDS}"  TZDB_LEAPSECONDS )

    if( WIN32  OR  MSVC  OR  MINGW  OR  CYGWIN )
        file( READ  "${DATE_INSTALL_DIR}/windowsZones.xml"  TZDB_WINDOWSZONES )
        string( REGEX  REPLACE  "\""  "\\\\\""  TZDB_WINDOWSZONES  ${TZDB_WINDOWSZONES} )
        string( REGEX  REPLACE  "([^\n]+)\n"  "\"\\1\\\\n\"\n"  TZDB_WINDOWSZONES  ${TZDB_WINDOWSZONES} )
    endif()

    configure_file( tz.cpp  tz.cpp  @ONLY )
endif()

# get a list of all headers
file( GLOB_RECURSE  _header_files  RELATIVE  ${CMAKE_CURRENT_LIST_DIR}  "${PROJECT_SOURCE_DIR}/include/*.h" )
# add the library target
add_library( date
    ${CMAKE_CURRENT_BINARY_DIR}/tz.cpp
    ${_header_files}  # for IDEs
)
unset( _header_files  CACHE )

target_include_directories( date
    PUBLIC   "$<INSTALL_INTERFACE:include>"
    PUBLIC   "$<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>"
    PRIVATE  "$<$<BOOL:${CURL_FOUND}>:${CURL_INCLUDE_DIRS}>"
)

# attempt to find a threading library
set( CMAKE_THREAD_PREFER_PTHREAD  TRUE )
set( THREADS_PREFER_PTHREAD_FLAG  TRUE )
find_package( Threads )

target_link_libraries( date
    PRIVATE  "$<$<BOOL:${CURL_FOUND}>:${CURL_LIBRARIES}>"
    PRIVATE  "$<$<BOOL:TARGET Threads>:Threads\:\:Threads>"
)

target_compile_definitions( date
    PRIVATE  "$<$<BOOL:DEFINED  DATE_INSTALL>:INSTALL=${DATE_INSTALL}>"
    PRIVATE  "$<$<BOOL:DEFINED  DATE_HAS_REMOTE_API>:HAS_REMOTE_API=$<BOOL:${DATE_HAS_REMOTE_API}>>"
    PRIVATE  "$<$<BOOL:DEFINED  DATE_AUTO_DOWNLOAD>:AUTO_DOWNLOAD=$<BOOL:${DATE_AUTO_DOWNLOAD}>>"
    PRIVATE  "$<$<BOOL:DEFINED  DATE_USE_OS_TZDB>:USE_OS_TZDB=$<BOOL:${DATE_USE_OS_TZDB}>>"
    PRIVATE  "$<$<BOOL:DEFINED  DATE_USE_SHELL_API>:USE_SHELL_API=$<BOOL:${DATE_USE_SHELL_API}>>"
    PRIVATE  "$<$<BOOL:DEFINED  DATE_MAKE_PORTABLE>:MAKE_PORTABLE=$<BOOL:${DATE_MAKE_PORTABLE}>>"
    PRIVATE  "$<$<BOOL:DEFINED  DATE_ONLY_C_LOCALE>:ONLY_C_LOCALE=$<BOOL:${DATE_ONLY_C_LOCALE}>>"
)
