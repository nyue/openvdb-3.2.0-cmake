CMAKE_MINIMUM_REQUIRED ( VERSION 2.8.11 )

PROJECT ( OpenVDB_Core )

MESSAGE ( "CMAKE_SOURCE_DIR = ${CMAKE_SOURCE_DIR}")
MESSAGE ( "CMAKE_BINARY_DIR = ${CMAKE_BINARY_DIR}")

FIND_PACKAGE ( GLFW3 REQUIRED )
FIND_PACKAGE ( OpenGL REQUIRED )
FIND_PACKAGE ( ILMBase REQUIRED )
FIND_PACKAGE ( OpenEXR REQUIRED )
FIND_PACKAGE ( TBB REQUIRED )
FIND_PACKAGE ( Boost 1.55 REQUIRED COMPONENTS system iostreams thread )

INCLUDE_DIRECTORIES ( ${GLFW3_INCLUDE_DIR} )
INCLUDE_DIRECTORIES ( ${ILMBASE_INCLUDE_DIRS} )
INCLUDE_DIRECTORIES ( ${TBB_INCLUDE_DIR} )
INCLUDE_DIRECTORIES ( ${Boost_INCLUDE_DIR} )

EXECUTE_PROCESS(
  COMMAND "${CMAKE_COMMAND}" "-E" "create_symlink" "${CMAKE_SOURCE_DIR}/viewer" "${CMAKE_BINARY_DIR}/openvdb_viewer"
  COMMAND "${CMAKE_COMMAND}" "-E" "create_symlink" "${CMAKE_SOURCE_DIR}" "${CMAKE_BINARY_DIR}/openvdb"
  RESULT_VARIABLE rc
  OUTPUT_VARIABLE output
  ERROR_VARIABLE error
)
INCLUDE_DIRECTORIES ( ${CMAKE_BINARY_DIR} ) # So that the embedded #include <openvdb/*> works!

ADD_DEFINITIONS (
  -fPIC
  -DOPENVDB_PRIVATE
  -std=c++0x
  # -pthread -O3 -DNDEBUG -I . -I .. -isystem /include -isystem /include -isystem /include -isystem /rel/folio/log4cplus/log4cplus-1.0.3-latest/sys_include -DOPENVDB_USE_LOG4CPLUS 
  -DOPENVDB_USE_GLFW_3
  -DGL_GLEXT_PROTOTYPES=1
  )

SET ( OPENVDB_LIBRARY_SOURCES
  Grid.cc
  io/Archive.cc
  io/Compression.cc
  io/File.cc
  io/GridDescriptor.cc
  io/Queue.cc
  io/Stream.cc
  io/TempFile.cc
  math/Maps.cc
  math/Proximity.cc
  math/QuantizedUnitVec.cc
  math/Transform.cc
  metadata/Metadata.cc
  metadata/MetaMap.cc
  openvdb.cc
  Platform.cc
  util/Formats.cc
  util/Util.cc
  )

ADD_LIBRARY ( openvdb_static STATIC
  ${OPENVDB_LIBRARY_SOURCES}
  )

ADD_LIBRARY ( openvdb_shared SHARED
  ${OPENVDB_LIBRARY_SOURCES}
  )

IF (WIN32)
  SET_TARGET_PROPERTIES ( openvdb_static
	PROPERTIES OUTPUT_NAME libopenvdb
	)
ELSE (WIN32)
  SET_TARGET_PROPERTIES ( openvdb_static
	PROPERTIES OUTPUT_NAME openvdb
	)
ENDIF ()

SET_TARGET_PROPERTIES ( openvdb_shared
  PROPERTIES OUTPUT_NAME openvdb
  )

# Applications
#  vdb_print           command-line tool to inspect OpenVDB files
#  vdb_render          command-line tool to ray-trace OpenVDB files
#  vdb_view            command-line tool to view OpenVDB files

ADD_EXECUTABLE ( vdb_print
  cmd/openvdb_print/main.cc 
  )

ADD_EXECUTABLE ( vdb_render
  cmd/openvdb_render/main.cc
  )

ADD_EXECUTABLE ( vdb_view
  viewer/Camera.cc
  viewer/ClipBox.cc
  viewer/Font.cc
  viewer/RenderModules.cc
  viewer/Viewer.cc
  cmd/openvdb_view/main.cc
  )

SET ( OPENVDB_APPLICATION_LINK_LIBRARIES
  openvdb_shared
  ${Tbb_TBB_LIBRARY}
  ${Ilmbase_HALF_LIBRARY}
  ${Openexr_ILMIMF_LIBRARY}
  ${Boost_LIBRARIES}
  )

SET ( OPENVDB_GLFW3_LINK_LIBRARIES
  ${GLFW3_glfw_LIBRARY}
  ${OPENGL_gl_LIBRARY}
  ${OPENGL_glu_LIBRARY}
  )

TARGET_LINK_LIBRARIES ( vdb_print
  ${OPENVDB_APPLICATION_LINK_LIBRARIES}
  )

TARGET_LINK_LIBRARIES ( vdb_render
  ${OPENVDB_APPLICATION_LINK_LIBRARIES}
  )

TARGET_LINK_LIBRARIES ( vdb_view
  ${OPENVDB_APPLICATION_LINK_LIBRARIES}
  ${OPENVDB_GLFW3_LINK_LIBRARIES}
  )
