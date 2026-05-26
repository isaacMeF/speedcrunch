PROJECT(speedcrunch)

# note: change this BEFORE making a release !!
SET( speedcrunch_VERSION "trunk" )

IF( APPLE )
  SET( PROGNAME SpeedCrunch )
  SET( MACOSX_BUNDLE_ICON_FILE speedcrunch.icns )
  SET( MACOSX_BUNDLE_SHORT_VERSION_STRING ${speedcrunch_VERSION} )
  SET( MACOSX_BUNDLE_VERSION ${speedcrunch_VERSION} )
  SET( MACOSX_BUNDLE_LONG_VERSION_STRING Version ${speedcrunch_VERSION} )
ELSE( APPLE )
  SET( PROGNAME speedcrunch )
ENDIF( APPLE )

cmake_minimum_required(VERSION 3.5)
SET(CMAKE_COLOR_MAKEFILE ON)
SET(CMAKE_VERBOSE_MAKEFILE OFF)
SET(CMAKE_INCLUDE_CURRENT_DIR TRUE)

ADD_DEFINITIONS("-DSPEEDCRUNCH_VERSION=\\\"${speedcrunch_VERSION}\\\"")

IF( NOT WIN32 )
IF( NOT APPLE )
  # for BinReloc stuff
  ADD_DEFINITIONS( -DENABLE_BINRELOC )
ENDIF( NOT APPLE )
ENDIF( NOT WIN32 )

INCLUDE_DIRECTORIES(${CMAKE_BINARY_DIR} 3rdparty/util base gui main math)

# sources
INCLUDE(SourceFiles.cmake)

# resources
SET(speedcrunch_RESOURCES resources/speedcrunch.qrc)

# translations
INCLUDE(Translations.cmake)

IF( MINGW )
  # resource compilation for mingw
  ADD_CUSTOM_COMMAND( OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/speedcrunchico.o
                      COMMAND windres.exe -I${CMAKE_CURRENT_SOURCE_DIR} -i${CMAKE_CURRENT_SOURCE_DIR}/resources/speedcrunchico.rc -o ${CMAKE_CURRENT_BINARY_DIR}/speedcrunchico.o )
  SET(speedcrunch_SOURCES ${speedcrunch_SOURCES} ${CMAKE_CURRENT_BINARY_DIR}/speedcrunchico.o)
  SET(LINK_FLAGS -Wl,-subsystem,windows)
ELSE(MINGW)
  SET(speedcrunch_SOURCES ${speedcrunch_SOURCES} resources/speedcrunchico.rc)
ENDIF(MINGW)

# enable warnings
IF(CMAKE_COMPILER_IS_GNUCXX )
  ADD_DEFINITIONS( -g -Wall )
ENDIF(CMAKE_COMPILER_IS_GNUCXX )

# setup for Qt5
SET(QT_USE_QTXML TRUE)
find_package(Qt5 COMPONENTS Core Widgets Gui Xml REQUIRED)
include_directories(${Qt5Core_INCLUDE_DIRS} ${Qt5Gui_INCLUDE_DIRS} ${Qt5Widgets_INCLUDE_DIRS} ${Qt5Xml_INCLUDE_DIRS})

# build everything
qt5_add_resources( speedcrunch_RESOURCES_SOURCES ${speedcrunch_RESOURCES} )
qt5_wrap_ui( speedcrunch_FORMS_HEADERS ${speedcrunch_FORMS} )
qt5_wrap_cpp( speedcrunch_HEADERS_MOC ${speedcrunch_HEADERS} )
find_program(QT_LRELEASE_EXECUTABLE lrelease)
IF( APPLE )
  ADD_EXECUTABLE( ${PROGNAME} MACOSX_BUNDLE ${speedcrunch_SOURCES} ${speedcrunch_HEADERS_MOC} ${speedcrunch_RESOURCES_SOURCES} ${speedcruch_FORMS_HEADERS} )
  SET( speedcrunch_RESOURCE_DIR ${CMAKE_CURRENT_BINARY_DIR}/${PROGNAME}.app/Contents/Resources )
  SET( speedcrunch_TRANSLATION_DIR ${CMAKE_CURRENT_BINARY_DIR}/${PROGNAME}.app/Contents/MacOS/locale )
  SET( speedcrunch_BOOKSDIR ${CMAKE_CURRENT_BINARY_DIR}/${PROGNAME}.app/Contents/MacOS/books )
  ADD_CUSTOM_COMMAND( TARGET ${PROGNAME} POST_BUILD
    COMMAND mkdir ARGS -p ${speedcrunch_RESOURCE_DIR}
    COMMAND mkdir ARGS -p ${speedcrunch_TRANSLATION_DIR}
    COMMAND mkdir ARGS -p ${speedcrunch_BOOKSDIR}
    COMMAND cp ARGS -f src/resources/${MACOSX_BUNDLE_ICON_FILE} ${speedcrunch_RESOURCE_DIR}
    COMMAND cp ARGS -f src/speedcrunch.desktop ${speedcrunch_RESOURCE_DIR}
    COMMAND bash -c "${QT_LRELEASE_EXECUTABLE} src/i18n/*.ts -qm src/i18n/*.qm"
    COMMAND cp ARGS -f src/i18n/*.qm ${speedcrunch_TRANSLATION_DIR}
    COMMAND cp ARGS -f src/resources/speedcrunch.png ${speedcrunch_RESOURCE_DIR}
    COMMAND cp ARGS -Rf src/books/ ${speedcrunch_BOOKSDIR}
    COMMAND rm ARGS -f ${speedcrunch_BOOKSDIR}/genbooks.sh
    COMMAND rm ARGS -rf ${speedcrunch_BOOKSDIR}/i18n
    COMMAND for ARGS svnDir in `find ${speedcrunch_BOOKSDIR} -type d -name \".svn\"` \; do rm -rf $svnDir \; done
    COMMENT Setting up translations & resources )
ELSE( APPLE )
  ADD_EXECUTABLE( ${PROGNAME} ${speedcrunch_SOURCES} ${speedcrunch_HEADERS_MOC} ${speedcrunch_RESOURCES_SOURCES} ${speedcrunch_FORMS_HEADERS} )
ENDIF( APPLE )

ADD_CUSTOM_TARGET( confclean COMMAND rm -rf Makefile CMakeFiles/ CMakeCache.txt cmake_install.cmake DartTestfile.txt install_manifest.txt )

target_link_libraries(${PROGNAME} ${Qt5Core_LIBRARIES} ${Qt5Gui_LIBRARIES} ${Qt5Widgets_LIBRARIES} ${Qt5Xml_LIBRARIES})

# install executable and translation files
INSTALL(TARGETS ${PROGNAME} DESTINATION bin)
INSTALL(FILES ${speedcrunch_TRANSLATIONS} DESTINATION share/speedcrunch/locale)

# required for regression tests
ENABLE_TESTING()

ADD_EXECUTABLE(testhmath ${testhmath_SOURCES})
ADD_TEST(testhmath ${EXECUTABLE_OUTPUT_PATH}/testhmath)

ADD_EXECUTABLE(testfloatnum ${testfloatnum_SOURCES})
ADD_TEST(testfloatnum ${EXECUTABLE_OUTPUT_PATH}/testfloatnum)

SET(BOOKSDIR "${CMAKE_INSTALL_PREFIX}/share/${PROGNAME}/books/")
ADD_DEFINITIONS("-DBOOKSDIR=\\\"${BOOKSDIR}\\\"")

INSTALL(FILES       ${CMAKE_CURRENT_SOURCE_DIR}/books/books.css
        DESTINATION ${BOOKSDIR}
)

INSTALL(DIRECTORY   ${CMAKE_CURRENT_SOURCE_DIR}/books/images ${CMAKE_CURRENT_SOURCE_DIR}/books/ca ${CMAKE_CURRENT_SOURCE_DIR}/books/cs ${CMAKE_CURRENT_SOURCE_DIR}/books/de ${CMAKE_CURRENT_SOURCE_DIR}/books/en ${CMAKE_CURRENT_SOURCE_DIR}/books/es ${CMAKE_CURRENT_SOURCE_DIR}/books/es_AR ${CMAKE_CURRENT_SOURCE_DIR}/books/eu ${CMAKE_CURRENT_SOURCE_DIR}/books/fi ${CMAKE_CURRENT_SOURCE_DIR}/books/fr ${CMAKE_CURRENT_SOURCE_DIR}/books/he ${CMAKE_CURRENT_SOURCE_DIR}/books/id ${CMAKE_CURRENT_SOURCE_DIR}/books/it ${CMAKE_CURRENT_SOURCE_DIR}/books/nb ${CMAKE_CURRENT_SOURCE_DIR}/books/nl ${CMAKE_CURRENT_SOURCE_DIR}/books/pt ${CMAKE_CURRENT_SOURCE_DIR}/books/pt_BR ${CMAKE_CURRENT_SOURCE_DIR}/books/pl ${CMAKE_CURRENT_SOURCE_DIR}/books/ro ${CMAKE_CURRENT_SOURCE_DIR}/books/ru ${CMAKE_CURRENT_SOURCE_DIR}/books/sv ${CMAKE_CURRENT_SOURCE_DIR}/books/tr ${CMAKE_CURRENT_SOURCE_DIR}/books/zh_CN
        DESTINATION "${BOOKSDIR}"
        PATTERN     ".svn" EXCLUDE
)

INSTALL(FILES       ${CMAKE_CURRENT_SOURCE_DIR}/speedcrunch.desktop
        DESTINATION "${CMAKE_INSTALL_PREFIX}/share/applications/"
)

INSTALL(FILES       ${CMAKE_CURRENT_SOURCE_DIR}/resources/speedcrunch.png
        DESTINATION "${CMAKE_INSTALL_PREFIX}/share/pixmaps/"
)