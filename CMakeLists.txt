cmake_minimum_required( VERSION 2.8.12 )

project( date_prj )

find_package( Threads REQUIRED )


option( USE_SYSTEM_TZ_DB "Use the operating system's timezone database" OFF )
option( USE_TZ_DB_IN_DOT "Save the timezone database in the current folder" OFF )
option( BUILD_TESTS "Build Tests" OFF )
option( RUN_TESTS "Build Tests" OFF )

if( USE_SYSTEM_TZ_DB ) 
	add_definitions( -DUSE_AUTOLOAD=0 )
	add_definitions( -DHAS_REMOTE_API=0 )
	add_definitions( -DUSE_OS_TZDB=1 )
else( )
	add_definitions( -DUSE_AUTOLOAD=1 )
	add_definitions( -DHAS_REMOTE_API=1 )
	add_definitions( -DUSE_OS_TZDB=0 )
	find_package( CURL REQUIRED )
	find_package( OpenSSL REQUIRED )
	include_directories( SYSTEM ${OPENSSL_INCLUDE_DIR} )
	include_directories( SYSTEM ${CURL_INCLUDE_DIRS} )
	set( OPTIONAL_LIBRARIES ${CURL_LIBRARIES} ${OPENSSL_LIBRARIES} )
endif( )

if( USE_TZ_DB_IN_DOT ) 
	add_definitions( -DINSTALL=. )
endif( )

if( RUN_TESTS )
	set( BUILD_TESTS ON )
endif( )

set( HEADER_FOLDER "include" )
set( SOURCE_FOLDER "src" )
set( TEST_FOLDER "test" )

include_directories( ${HEADER_FOLDER} )

# This is needed so IDE's live MSVC show header files
set( HEADER_FILES
	${HEADER_FOLDER}/date/chrono_io.h
	${HEADER_FOLDER}/date/date.h
	${HEADER_FOLDER}/date/ios.h
	${HEADER_FOLDER}/date/islamic.h
	${HEADER_FOLDER}/date/iso_week.h
	${HEADER_FOLDER}/date/julian.h
	${HEADER_FOLDER}/date/tz.h
	${HEADER_FOLDER}/date/tz_private.h
)

add_library( tz SHARED ${HEADER_FILES} ${SOURCE_FOLDER}/tz.cpp )
set_property(TARGET tz PROPERTY CXX_STANDARD 14)
target_link_libraries( tz ${CMAKE_THREAD_LIBS_INIT} ${OPTIONAL_LIBRARIES} ${CMAKE_THREAD_LIBS_INIT}  )

install( TARGETS tz DESTINATION lib )
install( DIRECTORY ${HEADER_FOLDER}/ DESTINATION include/ )

if( BUILD_TESTS )
	file( GLOB_RECURSE TEST_FILES ${TEST_FOLDER} *.cpp )
	foreach( CUR_TEST ${TEST_FILES} ) 
		MESSAGE(STATUS "Process file: ${CUR_TEST}")
	endforeach( CUR_TEST )
endif( )
