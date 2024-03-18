cmake_minimum_required (VERSION 3.13.0)

set(ARCH_FLAGS "AVX2") # BMI2 AVX2 AVX512
set(IS_SOLVER ON)

add_subdirectory(src/egtb/dictzip)

project(fairySF_egtb LANGUAGES CXX)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

if(ARCH_FLAGS)
	if (WIN32)
		set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -arch:${ARCH_FLAGS}")
	else()
		set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -march=native")
	endif()
endif()

include_directories(
	src
)

set(APP_SRC
	src/benchmark.cpp
	src/bitbase.cpp
	src/bitboard.cpp
	src/endgame.cpp
	src/evaluate.cpp
	src/main.cpp
	src/material.cpp
	src/misc.cpp
	src/movegen.cpp
	src/movepick.cpp
	src/pawns.cpp
	src/position.cpp
	src/psqt.cpp
	src/search.cpp
	src/thread.cpp
	src/timeman.cpp
	src/tt.cpp
	src/uci.cpp
	src/ucioption.cpp
	src/tune.cpp
	src/syzygy/tbprobe.cpp
	src/nnue/evaluate_nnue.cpp
	src/nnue/features/half_ka_v2.cpp
	src/partner.cpp
	src/parser.cpp
	src/piece.cpp
	src/variant.cpp
	src/xboard.cpp
	src/nnue/features/half_ka_v2_variants.cpp
)

set(EGTB_SRC 
	src/egtb/elements.cpp
	src/egtb/elements.h
	src/egtb/tb_reader.cpp
	src/egtb/tb_reader.h
	src/egtb/tb_idx.cpp
	src/egtb/tb_idx.h
)
SOURCE_GROUP ("EGTB" FILES ${EGTB_SRC})

add_executable(${PROJECT_NAME}
	${APP_SRC}
	${EGTB_SRC}
)

target_compile_definitions(${PROJECT_NAME}
	PRIVATE USE_POPCNT
	PRIVATE USE_PEXT
	PRIVATE USE_FAIRY_SF
	PRIVATE NNUE_EMBEDDING_OFF
)
if(IS_SOLVER)
	target_compile_definitions(${PROJECT_NAME}
		PRIVATE IS_SOLVER
	)
endif()

if (WIN32)
	target_link_libraries(${PROJECT_NAME}
		$<TARGET_OBJECTS:dictzip>
	)
else()
	target_compile_definitions(${PROJECT_NAME}
		PRIVATE NDEBUG
	)
	find_package(Threads REQUIRED)
	target_link_libraries(${PROJECT_NAME}
		$<TARGET_OBJECTS:dictzip>
		Threads::Threads
		-lstdc++fs
	)
endif()

if(WIN32 AND ARCH_FLAGS)
	string(TOLOWER ${ARCH_FLAGS} suff_arch)
	set(suff_arch "_${suff_arch}")
else()
	set(suff_arch "")
endif()

if(IS_SOLVER)
	set(suff_solver "")
else()
	set(suff_solver "_li")
endif()

set_target_properties(${PROJECT_NAME} PROPERTIES
	OUTPUT_NAME "${PROJECT_NAME}${suff_arch}${suff_solver}"
)

if (WIN32)
	target_compile_options(${PROJECT_NAME}
		PRIVATE "/Ox"
		PRIVATE "/GL"
	)
else()
	target_compile_options(${PROJECT_NAME}
		PRIVATE "-Ofast"
		PRIVATE "-flto"
		PRIVATE "-march=native"
	)
endif()
