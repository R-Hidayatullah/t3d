cmake_minimum_required (VERSION 3.2)

project (t3dtools)

# CMake 3.27+ may ask the linker to emit dependency files via
# --dependency-file. wasm-ld does not support that flag reliably, so
# disable linker-generated depfiles and let CMake track inputs itself.
set(CMAKE_LINK_DEPENDS_USE_LINKER FALSE)

SET(CMAKE_C_COMPILER emcc)
SET(CMAKE_CXX_COMPILER em++)
SET(CMAKE_AR emar)
SET(CMAKE_RANLIB emranlib)
add_definitions(-std=c++11)

include_directories(src/)
add_library(gw2DatTools STATIC 
	src/gw2DatTools/utils/BitArray.i
	src/gw2DatTools/compression/inflateTextureFileBuffer.cpp
	src/gw2DatTools/compression/huffmanTreeUtils.cpp
	src/gw2DatTools/compression/inflateDatFileBuffer.cpp
	src/gw2DatTools/compression/HuffmanTree.i
	src/gw2DatTools/exception/Exception.cpp
	src/gw2DatTools/format/ANDat.cpp
	src/gw2DatTools/format/Mapping.cpp
	src/gw2DatTools/format/Mft.cpp
	src/gw2DatTools/interface/ANDatInterface.cpp
)

add_library(gw2Formats STATIC
	src/gw2formats/TextureFile.cpp
)

add_library(libsquish STATIC
	src/squish/singlecolourfit.cpp
	src/squish/clusterfit.cpp 
	src/squish/colourset.cpp
	src/squish/rangefit.cpp 
	src/squish/maths.cpp 
	src/squish/texture_compression_s3tc.txt
	src/squish/squish-Info.plist
	src/squish/config 
	src/squish/colourfit.cpp
	src/squish/singlecolourlookup.inl
	src/squish/squish.cpp
	src/squish/colourblock.cpp
	src/squish/alpha.cpp
)

set(ALL_LINK_OPTIONS
	"--pre-js"
	"${CMAKE_CURRENT_SOURCE_DIR}/src/pre.js"
	"-sEXPORTED_FUNCTIONS=_inflate,_workImage,_malloc,_free"
	"-sEXPORTED_RUNTIME_METHODS=ccall,setValue,writeArrayToMemory,getValue"
	"-sALLOW_MEMORY_GROWTH=1"
	"-sNO_DISABLE_EXCEPTION_CATCHING"
	"-sASSERTIONS"
)

set(T3DTOOLS_LINK_OPTIONS
	${ALL_LINK_OPTIONS}
	"-sEXPORT_NAME=t3dtools"
	"-sMODULARIZE=1"
)

set(T3DWORKER_LINK_OPTIONS
	${ALL_LINK_OPTIONS}
	"--post-js"
	"${CMAKE_CURRENT_SOURCE_DIR}/src/t3dlib_worker.js"
	"-sWASM_ASYNC_COMPILATION=0"
)

add_executable(t3dtools.js src/t3dtools.cc)
target_sources(t3dtools.js PUBLIC src/3dcx.cc)
target_link_libraries(t3dtools.js gw2DatTools gw2Formats libsquish)
target_link_options(t3dtools.js PRIVATE ${T3DTOOLS_LINK_OPTIONS})
set_target_properties(t3dtools.js PROPERTIES ADDITIONAL_CLEAN_FILES t3dtools.wasm)

add_executable(t3dworker.js src/t3dtools.cc)
target_sources(t3dworker.js PUBLIC src/3dcx.cc)
target_link_libraries(t3dworker.js gw2DatTools gw2Formats libsquish)
target_link_options(t3dworker.js PRIVATE ${T3DWORKER_LINK_OPTIONS})
set_target_properties(t3dworker.js PROPERTIES ADDITIONAL_CLEAN_FILES t3dworker.wasm)
