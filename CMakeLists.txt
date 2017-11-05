cmake_minimum_required( VERSION  3.3 )

project( date
         VERSION    2.3.0
         LANGUAGES  CXX
)

set( CMAKE_CXX_STANDARD  14 )
set( CMAKE_CXX_STANDARD_REQUIRED  ON )
set_property( GLOBAL  PROPERTY  USE_FOLDERS  ON )

## INSTALL ##
set( DATE_INSTALL  ${CMAKE_BINARY_DIR}  CACHE  PATH
    "The directory where the uncompressed IANA Time Zone Database is located."
)
set( DATE_INSTALL_DIR  "${DATE_INSTALL}/tzdata" )

## HAS_REMOTE_API ##
set( _date_has_remote_api_default  OFF )
find_package( CURL )    # attempt to find curl
if( CURL_FOUND )
    set( _date_has_remote_api_default  ON )
endif()
option( DATE_HAS_REMOTE_API
    "When enabled, then the remote API exists. Otherwise it does not."
    ${_date_has_remote_api_default}
)
unset( _date_has_remote_api_default )

## AUTO_DOWNLAOD ##
set( _date_auto_download_default  OFF )
if( NOT  DATE_USE_OS_TZDB )
    set( _date_auto_download_default  ${DATE_HAS_REMOTE_API} )
endif()
option( DATE_AUTO_DOWNLOAD
    "When enabled, the first access to the timezone database will install it if it hasn't \
been installed, and if it has, will use the remote API to install the latest version if \
not already installed."
    ${_date_auto_download_default}
)
unset( _date_auto_download_default )

## USE_OS_TZDB ##
if( NOT  WIN32  AND  NOT  MSVC  AND  NOT  MINGW  AND  NOT  CYGWIN )
    option( DATE_USE_OS_TZDB
        "When enabled, this library will use the zic-compiled time zone database provided by \
your OS. This option relieves you of having to install the IANA time zone database, either \
manually, or automatically with DATE_AUTO_DOWNLOAD"
        OFF
    )
endif()

## USE_SHELL_API ##
option( DATE_USE_SHELL_API
    "When enabled, std::system is used to execute commands for downloading the timezone \
database. Otherwise, fork is used to execute commands for downloading the timezone database."
    ON
)

## ONLY_C_LOCALE ##
option( DATE_ONLY_C_LOCALE
    "Helps with systems with missing time_get/time_put facets."
    OFF
)

if( NOT  DEFINED  CMAKE_RUNTIME_OUTPUT_DIRECTORY )
  set( CMAKE_RUNTIME_OUTPUT_DIRECTORY  "${CMAKE_BINARY_DIR}/bin" )
endif()
if( NOT  DEFINED  CMAKE_LIBRARY_OUTPUT_DIRECTORY )
  set( CMAKE_LIBRARY_OUTPUT_DIRECTORY  "${CMAKE_BINARY_DIR}/lib" )
endif()
if( NOT  DEFINED  CMAKE_ARCHIVE_OUTPUT_DIRECTORY )
  set( CMAKE_ARCHIVE_OUTPUT_DIRECTORY  "${CMAKE_BINARY_DIR}/lib" )
endif()

enable_testing()

add_subdirectory( src )
add_subdirectory( test )
