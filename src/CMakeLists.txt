configure_file(${CMAKE_CURRENT_SOURCE_DIR}/global.h.in ${CMAKE_CURRENT_BINARY_DIR}/global.h)

include_directories(${CMAKE_CURRENT_BINARY_DIR})

set(liblastfm_SOURCES
        ws.cpp
        NetworkConnectionMonitor.cpp
        NetworkAccessManager.cpp
        InternetConnectionMonitor.cpp
        Xspf.cpp
        User.cpp
        Track.cpp
        Tasteometer.cpp
        Tag.cpp
        Playlist.cpp
        Mbid.cpp
        FingerprintId.cpp
        Artist.cpp
        Album.cpp
        ScrobbleCache.cpp
        ScrobblePoint.cpp
        Audioscrobbler.cpp
        RadioTuner.cpp
        RadioStation.cpp
        XmlQuery.cpp
        Url.cpp
        UrlBuilder.cpp
        misc.cpp
        Chart.cpp
        Auth.cpp
        Library.cpp
)

if(WIN32)
    add_definitions("-DWIN32_LEAN_AND_MEAN")

    if(NOT MINGW)
        add_definitions("-D_ATL_DLL -D_CRT_SECURE_NO_WARNINGS")

        set(liblastfm_SOURCES ${liblastfm_SOURCES}
            win/WNetworkConnectionMonitor_win.cpp
            win/WmiSink.cpp
            win/NdisEvents.cpp
        )
    endif()
endif()

if(APPLE)
    #FIXME: enable this when deploying
    #set(CMAKE_OSX_ARCHITECTURES "i386;ppc")
    #set(CMAKE_OSX_DEPLOYMENT_TARGET 10.5)
    #set(CMAKE_OSX_SYSROOT "/Developer/SDKs/MacOSX${CMAKE_OSX_DEPLOYMENT_TARGET}.sdk")

    set(liblastfm_SOURCES ${liblastfm_SOURCES}
        mac/MNetworkConnectionMonitor_mac.cpp
    )
endif()

if(UNIX AND NOT APPLE)
    set(liblastfm_SOURCES ${liblastfm_SOURCES}
        linux/LNetworkConnectionMonitor_linux.cpp
    )
endif()

add_library(${LASTFM_LIB_TARGET_NAME} SHARED ${liblastfm_SOURCES})

target_link_libraries(${LASTFM_LIB_TARGET_NAME} Qt::Network Qt::Xml)
if(UNIX AND NOT APPLE)
    target_link_libraries(${LASTFM_LIB_TARGET_NAME} Qt::DBus)
endif()

if(WIN32 AND NOT MINGW)
    target_link_libraries(${LASTFM_LIB_TARGET_NAME}
        winhttp
        wbemuuid
    )
endif()

if(APPLE)
    find_library(SYSTEMCONFIGURATION_LIBRARY SystemConfiguration)
    find_library(COREFOUNDATION_LIBRARY CoreFoundation)
    target_link_libraries(${LASTFM_LIB_TARGET_NAME}
        ${COREFOUNDATION_LIBRARY}
        ${SYSTEMCONFIGURATION_LIBRARY}
    )
endif()

set_target_properties(${LASTFM_LIB_TARGET_NAME} PROPERTIES
    VERSION ${LASTFM_VERSION_STRING}
    SOVERSION ${LASTFM_SOVERSION}
    COMPILE_DEFINITIONS LASTFM_LIB
)

if(WIN32)
    set_target_properties(${LASTFM_LIB_TARGET_NAME} PROPERTIES
        RUNTIME_OUTPUT_NAME "${LASTFM_LIB_TARGET_NAME}-${LASTFM_SOVERSION}")
endif()

install(TARGETS ${LASTFM_LIB_TARGET_NAME}
    EXPORT "${LASTFM_LIB_TARGET_NAME}Config"
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
)

install(EXPORT "${LASTFM_LIB_TARGET_NAME}Config"
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/lastfm${LASTFM_LIB_VERSION_SUFFIX}
    NAMESPACE LASTFM::
)

file(GLOB liblastfm_HEADERS ${CMAKE_CURRENT_LIST_DIR}/*.h)
list(APPEND liblastfm_HEADERS ${CMAKE_CURRENT_BINARY_DIR}/global.h)
install(FILES ${liblastfm_HEADERS} DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/lastfm${LASTFM_LIB_VERSION_SUFFIX}/)
