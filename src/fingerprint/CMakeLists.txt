include_directories(${CMAKE_CURRENT_LIST_DIR}/..)
include_directories(${CMAKE_CURRENT_BINARY_DIR}/..)

set(lastfm_fingerprint_SOURCES
  Collection.cpp
  Fingerprint.cpp
  Sha256.cpp
  fplib/Filter.cpp
  fplib/FingerprintExtractor.cpp
  fplib/OptFFT.cpp
)

set(lastfm_fingerprint_HEADERS
    Fingerprint.h
    FingerprintableSource.h
)

add_library(${FINGERPRINT_LIB_TARGET_NAME} SHARED ${lastfm_fingerprint_SOURCES})

set_target_properties(${FINGERPRINT_LIB_TARGET_NAME} PROPERTIES
    COMPILE_DEFINITIONS LASTFM_FINGERPRINT_LIB
    VERSION ${LASTFM_VERSION_STRING}
    SOVERSION ${LASTFM_SOVERSION}
)

if(WIN32)
    set_target_properties(${FINGERPRINT_LIB_TARGET_NAME} PROPERTIES
        RUNTIME_OUTPUT_NAME "${FINGERPRINT_LIB_TARGET_NAME}-${LASTFM_SOVERSION}")
endif()

target_link_libraries(${FINGERPRINT_LIB_TARGET_NAME}
    ${LASTFM_LIB_TARGET_NAME}
    PkgConfig::LibSamplerate
    PkgConfig::FFTW3F
)

target_link_libraries(${FINGERPRINT_LIB_TARGET_NAME} Qt::Network Qt::Sql Qt::Xml)

install(TARGETS ${FINGERPRINT_LIB_TARGET_NAME}
    EXPORT "${FINGERPRINT_LIB_TARGET_NAME}Config"
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
)

install(EXPORT "${FINGERPRINT_LIB_TARGET_NAME}Config"
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/lastfm${LASTFM_LIB_VERSION_SUFFIX}
    NAMESPACE LASTFM::
)

install(FILES ${lastfm_fingerprint_HEADERS} DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/lastfm${LASTFM_LIB_VERSION_SUFFIX}/)
