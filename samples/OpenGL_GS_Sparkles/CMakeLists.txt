Project(OpenGL_GS_Sparkles)

set(PLATFORM_LIBRARIES
    ${NVFX_LIBS}
    ${OPENGL_LIBRARY}
    ${GLEW_LIBRARY}
    ${ZLIB_LIBRARIES}
    ${CUDA_LIBRARIES}
)
include_directories(
    ${nvFx_SOURCE_DIR}/include
    ${GLEW_INCLUDE_DIR}
    ${ZLIB_INCLUDE_DIRS}
    ${SHARED_DIR}
    ${NV_MATH_DIR}
    ${BK3D_DIR}
    ${NV_IMAGE_DIR}
)
set(SOURCE_FILES 
  OpenGL_GS_Sparkles.cpp
  ${SHARED_DIR}/main.cpp
  OpenGL_GS_Sparkles.glslfx
  materialCarPaint.glslfx
)
#####################################################################################
# UI
if(SVCMFCUI_FOUND AND USE_SVCUI)
  include_directories(${SVCMFCUI_INCLUDE_DIR})
  LIST(APPEND SOURCE_FILES ${SVCMFCUI_HEADERS})
endif()
#####################################################################################
# Glew : source or lib
if(GLEW_SOURCE)
  message(STATUS "found Glew source code. Using it instead of library")
  LIST(APPEND SOURCE_FILES ${GLEW_SOURCE})
  add_definitions(-DGLEW_STATIC)
else()
    message(STATUS "using GLEW library")
    LIST(APPEND PLATFORM_LIBRARIES ${GLEW_LIBRARY})
endif()
add_definitions(-DGLEW_NO_GLU)
#####################################################################################
# GLUT
if(USE_GLUT AND GLUT_FOUND)
  LIST(APPEND PLATFORM_LIBRARIES ${GLUT_LIBRARIES} )
  include_directories(${GLUT_INCLUDE_DIR})
endif()
set(PUBLIC_HEADERS
  ${nvFx_SOURCE_DIR}/include/FxLib.h
  ${nvFx_SOURCE_DIR}/include/FxLibEx.h
  ${nvFx_SOURCE_DIR}/include/FxParser.h
)
source_group(nvFx FILES
  ${PUBLIC_HEADERS}
)
source_group(nv_math FILES
  ${NV_MATH_SOURCE}
  ${NV_MATH_HEADERS}
)
source_group(nvImage FILES
  ${NV_IMAGE_SOURCES}
  ${NV_IMAGE_HEADERS}
)
if(SVCMFCUI_FOUND AND USE_SVCUI)
  source_group(SvcUI FILES 
    ${SVCMFCUI_HEADERS}
  )
endif()
source_group(bk3d_headers FILES
  ${BK3D_HEADERS}
)

#if(APPLE)
#    _add_glut_executable(OpenGL_GS_Sparkles OpenGL_GS_Sparkles.cpp)
#    add_definitions(-DAPPLE)
#else()
#_add_glut_executable(OpenGL_GS_Sparkles
#    OpenGL_GS_Sparkles.cpp
#    ${INC_FILES}
#)
#endif()

if(NOT USE_GLUT)
  set(WIN_SRC
    ${SHARED_DIR}/resources.h
    ${SHARED_DIR}/resources.rc
  )
endif()

if(NVFXCC_ON_EFFECTS)
  add_custom_command(
    OUTPUT OpenGL_GS_Sparkles_Log.txt
    COMMAND ${NVFX_BINARY_PATH}/nvFxcc
            -oOpenGL_GS_Sparkles_Log.txt
            OpenGL_GS_Sparkles.glslfx
    MAIN_DEPENDENCY OpenGL_GS_Sparkles.glslfx
    DEPENDS ${NVFX_BINARY_PATH}/nvFxcc
    WORKING_DIRECTORY ${OpenGL_GS_Sparkles_SOURCE_DIR}
    )
  add_custom_command(
    OUTPUT materialCarPaint_Log.txt
    COMMAND ${NVFX_BINARY_PATH}/nvFxcc
            -omaterialCarPaint_Log.txt
            materialCarPaint.glslfx
    MAIN_DEPENDENCY materialCarPaint.glslfx
    DEPENDS ${NVFX_BINARY_PATH}/nvFxcc
    WORKING_DIRECTORY ${OpenGL_GS_Sparkles_SOURCE_DIR}
    )
  add_custom_target(EFFECTS_OpenGL_GS_Sparkles ALL DEPENDS OpenGL_GS_Sparkles_Log.txt materialCarPaint_Log.txt)
endif()

set(SRC_FOR_EXCUTABLE
  ${SOURCE_FILES}
  ${PUBLIC_HEADERS}
  ${SOURCES}
  ${NV_MATH_SOURCE}
  ${NV_MATH_HEADERS}
  ${NV_IMAGE_SOURCES}
  ${NV_IMAGE_HEADERS}
  ${BK3D_HEADERS}
  ${WIN_SRC}
  ${SHARED_DIR}/Camera.h
  ${SHARED_DIR}/Realtime.h
)

if(WIN32)
  add_executable(OpenGL_GS_Sparkles WIN32 ${SRC_FOR_EXCUTABLE} )
  if((NOT USE_GLUT) OR (NOT GLUT_FOUND))
    # strange: not working... but WIN32 above solve the issue
    #set_target_properties(OpenGL_GS_Sparkles PROPERTIES LINK_FLAGS_RELEASE "/SUBSYSTEM:WINDOWS")
    #set_target_properties(OpenGL_GS_Sparkles PROPERTIES LINK_FLAGS_DEBUG "/SUBSYSTEM:WINDOWS")
    add_definitions(-DNOGLUT)
  endif()
else()
  add_executable(OpenGL_GS_Sparkles ${SRC_FOR_EXCUTABLE} )
endif()
target_link_libraries(OpenGL_GS_Sparkles
    ${PLATFORM_LIBRARIES}
)

# if we wanted to have the effect close to the exe
# but let's use the effect from the resource directory instead : 
# the sample will pull the effect from RESOURCE_DIR
#_copy_files_to_target(OpenGL_GS_Sparkles ${PROJECT_SOURCE_DIR}/effect.glslfx)

if(USE_SVCUI AND SVCMFCUI_FOUND)
  # Copy the resources where the executable file will be created
  _copy_files_to_target( OpenGL_GS_Sparkles "${SVCMFCUI_DLL}")
endif()
if(ZLIB_FOUND)
  _copy_files_to_target( OpenGL_GS_Sparkles "${ZLIB_BIN}")
endif()
#_copy_files_to_target( simpleOpenGL "${RESOURCE_DIR}/NV_Shaderball_v133.bk3d.gz")
add_definitions(-DRESOURCE_DIR="${RESOURCE_DIR}" -DSOURCE_DIR="${OpenGL_GS_Sparkles_SOURCE_DIR}")
