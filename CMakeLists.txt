cmake_minimum_required(VERSION 3.0)

set(APP_NAME MyGame)
project (${APP_NAME})

option(USE_CHIPMUNK "Use chipmunk for physics library" ON)
option(USE_BOX2D "Use box2d for physics library" OFF)
option(DEBUG_MODE "Debug or release?" ON)

if(DEBUG_MODE)
  set(CMAKE_BUILD_TYPE DEBUG)
else(DEBUG_MODE)
  set(CMAKE_BUILD_TYPE RELEASE)
endif(DEBUG_MODE)

if (MSVC)
	set(CMAKE_C_FLAGS_DEBUG "-DCOCOS2D_DEBUG=1")
	set(CMAKE_CXX_FLAGS_DEBUG ${CMAKE_C_FLAGS_DEBUG})

	set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -D_CRT_SECURE_NO_WARNINGS")
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -D_CRT_SECURE_NO_WARNINGS")

elseif (CMAKE_COMPILER_IS_GNUCC OR CMAKE_COMPILER_IS_GNUCXX)
	set(CMAKE_C_FLAGS_DEBUG "-g -Wall -DCOCOS2D_DEBUG=1")
	set(CMAKE_CXX_FLAGS_DEBUG ${CMAKE_C_FLAGS_DEBUG})

        set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -std=c99")
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")

endif()

if(USE_CHIPMUNK)
  message("Using chipmunk ...")
  add_definitions(-DCC_ENABLE_CHIPMUNK_INTEGRATION=1)
  if(UNIX) # assuming linux
    add_definitions(-DLINUX )
  endif()
elseif(USE_BOX2D)
  message("Using box2d ...")
  add_definitions(-DCC_ENABLE_BOX2D_INTEGRATION=1)
  if(UNIX) # assuming linux
    add_definitions(-DLINUX )
  endif()
else(USE_CHIPMUNK)
  message(FATAL_ERROR "Must choose a physics library.")
endif(USE_CHIPMUNK)

# architecture
if ( CMAKE_SIZEOF_VOID_P EQUAL 8 )
set(ARCH_DIR "64-bit")
set(MINGW_EXT "mingw64")
else()
set(MINGW_EXT "mingw32")
set(ARCH_DIR "32-bit")
endif()

if( UNIX ) #assume linux
set(GAME_SRC
  proj.linux/main.cpp
  Classes/AppDelegate.cpp
  Classes/HelloWorldScene.cpp
)
elseif ( WIN32 )

ADD_DEFINITIONS (-D_USRDLL -DCOCOS2DXWIN32_EXPORTS -D_WINDOWS -DWIN32)

set(GAME_SRC
  proj.win32/main.cpp
  proj.win32/main.h
  proj.win32/resource.h
  Classes/AppDelegate.cpp
  Classes/HelloWorldScene.cpp
)
endif()

if ( WIN32 )

  add_executable(
    ${APP_NAME}
    WIN32
    ${GAME_SRC}
  )

else()

  add_executable(
    ${APP_NAME}
    ${GAME_SRC}
  )

endif()

if ( CMAKE_SIZEOF_VOID_P EQUAL 8 )
set(FMOD_LIB "fmodex64")
else()
set(FMOD_LIB "fmodex")
endif()

target_link_libraries(${APP_NAME}
  spine
  cocostudio
  cocosbuilder
  extensions
  audio
  cocos2d
  opengl32
  glfw3
  glew32
  ws2_32
  freetype
  png
  tiff
  jpeg
  webp
  tinyxml2
  unzip
  chipmunk
  xxhash
  unzip
  z
  )

get_filename_component(MY_CMAKE_DIRECTORY "${CMAKE_COMMAND}" DIRECTORY)


add_custom_command(
  TARGET ${APP_NAME}
  POST_BUILD
  COMMAND ${CMAKE_COMMAND} -E remove_directory ${CMAKE_BINARY_DIR}/Resources
  COMMAND ${CMAKE_COMMAND} -E copy_directory ${CMAKE_CURRENT_SOURCE_DIR}/Resources ${CMAKE_BINARY_DIR}/
)

if ( MINGW )

  # create a list of files to copy
  set( THIRD_PARTY_DLLS
    glew32.dll
    zlib1.dll
    glfw3.dll
    libwinpthread-1.dll
    libfreetype-6.dll
    libgcc_s_seh-1.dll
    libstdc++-6.dll
    libjpeg-8.dll
    libpng16-16.dll
    libtiff-5.dll
    libtinyxml2.dll
    libwebp-5.dll
    liblzma-5.dll
    libchipmunk.dll
  )


  foreach( file_i ${THIRD_PARTY_DLLS})
    add_custom_command(
      TARGET ${APP_NAME}
      POST_BUILD
      COMMAND ${CMAKE_COMMAND} -E copy ${MY_CMAKE_DIRECTORY}/../../${MINGW_EXT}/bin/${file_i} ${CMAKE_BINARY_DIR}/${file_i}
    )
  endforeach( file_i )

endif()
