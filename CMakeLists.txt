project(asio VERSION 1.30.2)

aux_source_directory(${PROJECT_SOURCE_DIR}/asio/src src)
add_library(${PROJECT_NAME} SHARED ${src})


target_include_directories(asio PUBLIC 
    ${PROJECT_SOURCE_DIR}/asio/include
    ${THIRD_PARTY_INCLUDE}/openssl-cmake/include
)

target_link_libraries(${PROJECT_NAME} 
    PRIVATE
    ssl
)

target_compile_definitions(asio PUBLIC
  ASIO_SEPARATE_COMPILATION
  ASIO_STANDALONE 
  ASIO_NO_DEPRECATED
  ASIO_DYN_LINK
  # to disable exception, add ASIO_NOEXCEPT
)

if(CMAKE_SYSTEM_NAME MATCHES "Windows")
    # macro see @ https://stackoverflow.com/a/40217291/1746503
    macro(get_win32_winnt version)
      if(CMAKE_SYSTEM_VERSION)
        set(ver ${CMAKE_SYSTEM_VERSION})
        string(REGEX MATCH "^([0-9]+).([0-9])" ver ${ver})
        string(REGEX MATCH "^([0-9]+)" verMajor ${ver})
        # Check for Windows 10, b/c we'll need to convert to hex 'A'.
        if("${verMajor}" MATCHES "10")
          set(verMajor "A")
          string(REGEX REPLACE "^([0-9]+)" ${verMajor} ver ${ver})
        endif("${verMajor}" MATCHES "10")
        # Remove all remaining '.' characters.
        string(REPLACE "." "" ver ${ver})
        # Prepend each digit with a zero.
        string(REGEX REPLACE "([0-9A-Z])" "0\\1" ver ${ver})
        set(${version} "0x${ver}")
      endif()
    endmacro()

    if(NOT DEFINED _WIN32_WINNT)
      get_win32_winnt(ver)
      set(_WIN32_WINNT ${ver})
    endif()

    message(STATUS "Set _WIN32_WINNET=${_WIN32_WINNT}")

    target_compile_definitions(asio PUBLIC _WIN32_WINNT=${_WIN32_WINNT} WIN32_LEAN_AND_MEAN)
endif()