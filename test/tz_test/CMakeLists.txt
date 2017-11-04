set( _tzdata_txt_filename  "tzdata${DATE_TZDB_VERSION}.txt" )
set( _tzdata_zip_filename  "${CMAKE_CURRENT_SOURCE_DIR}/${_tzdata_txt_filename}.zip" )

if( NOT  EXISTS  ${_tzdata_zip_filename} )
    message( WARNING  ""
"Testbench data for validating timezones does not exist for version ${DATE_TZDB_VERSION}."
"    Skipping test creation for timezone validation."
)
else()
    # extract the data we check our output against
    set( _tzdata_txt_filepath  "${CMAKE_CURRENT_BINARY_DIR}/${_tzdata_txt_filename}" )
    if( NOT  EXISTS  ${_tzdata_txt_filepath} )
        execute_process(
            COMMAND  ${CMAKE_COMMAND} -E tar xz ${_tzdata_zip_filename}
            WORKING_DIRECTORY  ${CMAKE_CURRENT_BINARY_DIR}
        )
        # convert to native line endings
        file( READ   ${_tzdata_txt_filepath}  _tzdata_txt_string )
        file( WRITE  ${_tzdata_txt_filepath}  ${_tzdata_txt_string} )
        unset( _tzdata_txt_string  CACHE )
    endif()
    unset( _tzdata_txt_filepath )

    # add a target for the validate binary
    add_executable( tz_validate  EXCLUDE_FROM_ALL
        validate.cpp
    )
    target_link_libraries( tz_validate
        PRIVATE  date
    )
    set_target_properties( tz_validate  PROPERTIES
        FOLDER  "Tests"
    )

    # add a custom command that will run the validate binary after building it
    get_filename_component( _tzdata_name  ${_tzdata_txt_filename}  NAME_WE )
    set( _tz_validate_output_filename  "${_tzdata_name}_output.txt" )
    unset( _tzdata_name )
    add_custom_target( tz_validate__run
        COMMAND  "$<TARGET_FILE:tz_validate>" > ${_tz_validate_output_filename}
        BYPRODUCTS  ${_tz_validate_output_filename}
        WORKING_DIRECTORY  ${CMAKE_CURRENT_BINARY_DIR}
    )
    add_dependencies( tz_validate__run
        tz_validate
    )
    set_target_properties( tz_validate__run  PROPERTIES
        FOLDER  "Tests"
    )

    add_test(
        NAME  tz_test__validate__build_and_run
        COMMAND  ${CMAKE_COMMAND} --build ${CMAKE_BINARY_DIR} --target tz_validate__run  # use CMAKE_CFG_INTDIR here?
        WORKING_DIRECTORY  ${CMAKE_BINARY_DIR}
    )

    add_test(
        NAME  tz_test__validate__compare
        COMMAND  ${CMAKE_COMMAND} -E compare_files ${_tzdata_txt_filename} ${_tz_validate_output_filename}
        WORKING_DIRECTORY  ${CMAKE_CURRENT_BINARY_DIR}
    )
    unset( _tz_validate_output_filename )

    set_tests_properties( tz_test__validate__compare  PROPERTIES
        DEPENDS  tz_test__validate__build_and_run
    )
endif()

unset( _tzdata_txt_filename )
unset( _tzdata_zip_filename )
