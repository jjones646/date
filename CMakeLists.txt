cmake_minimum_required( VERSION  3.3 )

project( date
         VERSION    2.3.0
         LANGUAGES  CXX
)

set( CMAKE_CXX_STANDARD  14 )
set( CMAKE_CXX_STANDARD_REQUIRED  ON )
set_property( GLOBAL  PROPERTY  USE_FOLDERS  ON )

## MAKE_PORTABLE ##
option( DATE_MAKE_PORTABLE
    "When enabled, builds the timezone database into the library."
    OFF
)

## INSTALL ##
# we'll always define this cache variable, but won't assume it's a valid value
set( _date_install_default  ${CMAKE_BINARY_DIR} )
if( DATE_MAKE_PORTABLE )
    set( _date_install_default  "X" )
endif()
set( DATE_INSTALL  ${_date_install_default}  CACHE  PATH
    "The directory where the uncompressed IANA Time Zone Database is located."
)
unset( _date_install_default )
# full directory path to where the timezone database files get placed, assumed to always be valid
set( _date_install_root  ${DATA_INSTALL} )
if( DATE_MAKE_PORTABLE )
    set( _date_install_root  ${CMAKE_BINARY_DIR} )
endif()
set( DATE_INSTALL_DIR  "${_date_install_root}/tzdata" )
unset( _date_install_root )

## HAS_REMOTE_API ##
set( _date_has_remote_api_default  OFF )
find_package( CURL )    # attempt to find curl
if( CURL_FOUND )
    if( NOT  DATE_MAKE_PORTABLE )
        set( _date_has_remote_api_default  ON )
    endif()
endif()
option( DATE_HAS_REMOTE_API
    "When enabled, then the remote API exists. Otherwise it does not."
    ${_date_has_remote_api_default}
)
unset( _date_has_remote_api_default )

## AUTO_DOWNLAOD ##
set( _date_auto_download_default  OFF )
if( NOT  DATE_USE_OS_TZDB )
    if( NOT  DATE_MAKE_PORTABLE )
        set( _date_auto_download_default  ${DATE_HAS_REMOTE_API} )
    endif()
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

if( DATE_MAKE_PORTABLE )
    # get all available versions of the IANA Time Zone Database
    set( _tzdb_versions_filepath  "${CMAKE_CURRENT_BINARY_DIR}/time-zone-releases.html" )
    if( NOT  EXISTS  ${_tzdb_versions_filepath} )
        file(
            DOWNLOAD  "ftp://ftp.iana.org/tz/releases/"  ${_tzdb_versions_filepath}
            STATUS  _download_status
        )
        # check for download errors
        list( GET  _download_status  0  _download_status_code )
        if( NOT  ${_download_status_code}  EQUAL  0 )
            file( REMOVE  ${_tzdb_versions_filepath} )
            list( GET  _download_status  1  _download_status_message )
            message( FATAL_ERROR  "Unable to download list of IANA Time Zone Database versions: ${_download_status_message}" )
        endif()
        unset( _download_status )
        unset( _download_status_code )
    endif()
    # filter the raw text - creating a sorted list of versions
    file( STRINGS  ${_tzdb_versions_filepath}  _tzdb_raw_versions_list  REGEX  "tzdata[0-9][0-9][0-9][0-9][a-z]\." )
    unset( _tzdb_versions_filepath )
    set( _tzdb_versions_list )
    foreach( _raw_version  ${_tzdb_raw_versions_list} )
        string( REGEX  MATCH  "tzdata([0-9][0-9][0-9][0-9][a-z])\."  _version  ${_raw_version} )
        list( APPEND  _tzdb_versions_list  ${CMAKE_MATCH_1} )
    endforeach()
    unset( _tzdb_raw_versions_list  CACHE )
    list( REMOVE_DUPLICATES  _tzdb_versions_list )
    list( SORT  _tzdb_versions_list )
    # get the latest version
    list( LENGTH  _tzdb_versions_list  _tzdb_versions_list_end )
    math( EXPR  _tzdb_versions_list_end  "${_tzdb_versions_list_end} - 1" )
    list( GET  _tzdb_versions_list  ${_tzdb_versions_list_end}  _tzdb_latest_version )
    unset( _tzdb_versions_list_end )

    set( DATE_TZDB_VERSION  ${_tzdb_latest_version}  CACHE  STRING
        "The version of the IANA Time Zone Database to build into the library when \
DATE_MAKE_PORTABLE is enabled."
    )
    unset( _tzdb_latest_version )
    set_property( CACHE  DATE_TZDB_VERSION  PROPERTY  STRINGS  ${_tzdb_versions_list} )
    unset( _tzdb_versions_list )

    # make sure directory exists
    file( MAKE_DIRECTORY  ${DATE_INSTALL_DIR} )

    if( WIN32  OR  MSVC  OR  MINGW  OR  CYGWIN )
        # path to where windows zones is placed
        set( _windows_zones_filepath  "${DATE_INSTALL_DIR}/windowsZones.xml" )
        # download the file if it does not exist
        if( NOT  EXISTS  ${_windows_zones_filepath} )
            file(
                DOWNLOAD  "https://unicode.org/repos/cldr/trunk/common/supplemental/windowsZones.xml"  ${_windows_zones_filepath}
                TLS_VERIFY  ON
                TIMEOUT  10
                STATUS  _download_status
            )
            list( GET  _download_status  0  _download_status_code )
            if( NOT  ${_download_status_code}  EQUAL  0 )
                file( REMOVE  ${_windows_zones_filepath} )
                list( GET  _download_status  1  _download_status_message )
                message( FATAL_ERROR  "Unable to download windowsZones.xml: ${_download_status_message}" )
            endif()
            unset( _download_status )
            unset( _download_status_code )
        endif()
        unset( _windows_zones_filepath )
    endif()

    # download and extract the timezone database
    if( NOT  EXISTS  "${DATE_INSTALL_DIR}/README" )
        get_filename_component( _tzdb_parent_dir  ${DATE_INSTALL_DIR}  DIRECTORY )
        set( _tzdb_archive_filepath  "${_tzdb_parent_dir}/tzdata${DATE_TZDB_VERSION}.tar.gz" )
        unset( _tzdb_parent_dir )
        get_filename_component( _tzdb_archive_filename  ${_tzdb_archive_filepath}  NAME )

        file(
           DOWNLOAD  "https://www.iana.org/time-zones/repository/releases/${_tzdb_archive_filename}"  ${_tzdb_archive_filepath}
           TLS_VERIFY  ON
           TIMEOUT  10
           STATUS  _download_status
        )
        list( GET  _download_status  0  _download_status_code )
        if( NOT  ${_download_status_code}  EQUAL  0 )
            file( REMOVE  ${_tzdb_archive_filepath} )
            list( GET  _download_status  1  _download_status_message )
            message( FATAL_ERROR  "Unable to download ${_tzdb_archive_filename}: ${_download_status_message}" )
        endif()
        unset( _download_status )
        unset( _download_status_code )

        execute_process(
            COMMAND  ${CMAKE_COMMAND} -E tar xf ${_tzdb_archive_filepath}
            WORKING_DIRECTORY  ${DATE_INSTALL_DIR}
        )

        unset( _tzdb_archive_filepath )
        unset( _tzdb_archive_filename )
    endif()
endif()

enable_testing()

add_subdirectory( src )
add_subdirectory( test )
