
cmake_minimum_required(VERSION 3.18)

project(gflip3d LANGUAGES CXX CUDA)

# Set default build type to Release if not specified
if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Release)
endif()

# Set C++ and CUDA standards
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CUDA_STANDARD 20)
set(CMAKE_CUDA_STANDARD_REQUIRED ON)

# Modern CMake: Use file(GLOB_RECURSE) or explicitly list files
set(SOURCES
    GDelFlipping/src/DelaunayChecker.cu
    GDelFlipping/src/Demo.cu
    GDelFlipping/src/InputCreator.cu
    GDelFlipping/src/RandGen.cpp
    GDelFlipping/src/gDel3D/GpuDelaunay.cu
    GDelFlipping/src/gDel3D/CPU/predicates.cpp
    GDelFlipping/src/gDel3D/CPU/PredWrapper.cu
    GDelFlipping/src/gDel3D/CPU/Splaying.cu
    GDelFlipping/src/gDel3D/CPU/Star.cu
    GDelFlipping/src/gDel3D/GPU/KerDivision.cu
    GDelFlipping/src/gDel3D/GPU/KerPredicates.cu
    GDelFlipping/src/gDel3D/GPU/ThrustWrapper.cu
)

# Create executable target
add_executable(${PROJECT_NAME} ${SOURCES})

# Set target properties using modern CMake approach
set_target_properties(${PROJECT_NAME} PROPERTIES
    CUDA_SEPARABLE_COMPILATION ON
    CUDA_ARCHITECTURES "75;80;86;89"
    CXX_STANDARD 20
    CXX_STANDARD_REQUIRED ON
    CUDA_STANDARD 20
    CUDA_STANDARD_REQUIRED ON
)

# Modern CMake: Use target_include_directories with proper scope
target_include_directories(${PROJECT_NAME} PRIVATE 
    ${CMAKE_CUDA_TOOLKIT_INCLUDE_DIRECTORIES}
    GDelFlipping/src
)

# Set compiler-specific options
target_compile_options(${PROJECT_NAME} PRIVATE
    $<$<COMPILE_LANGUAGE:CUDA>:--extended-lambda>
    $<$<COMPILE_LANGUAGE:CUDA>:--expt-relaxed-constexpr>
    $<$<COMPILE_LANGUAGE:CUDA>:-Wno-deprecated-gpu-targets>
)

# Set optimization flags for Release build
target_compile_options(${PROJECT_NAME} PRIVATE
    $<$<CONFIG:Release>:-O3>
    $<$<CONFIG:Release>:$<$<COMPILE_LANGUAGE:CUDA>:-O3>>
)

# Fix library conflicts
if(WIN32)
    set_target_properties(${PROJECT_NAME} PROPERTIES
        LINK_FLAGS "/NODEFAULTLIB:LIBCMT"
    )
endif()
