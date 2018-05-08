
if("${CMAKE_CXX_COMPILER_ID}" MATCHES "Clang")
  set(CLANG_BIN ${CMAKE_CXX_COMPILER})
else()
  find_program(CLANG_BIN clang++)
endif()

find_program(LLVM_LINK_BIN llvm-link)

add_custom_command(OUTPUT libjit.bc libjit_conv.bc libjit_matmul.bc
  COMMAND ${CLANG_BIN} -std=c++11 -ffast-math -g -emit-llvm -c ${CMAKE_CURRENT_SOURCE_DIR}/libjit/libjit.cpp ${CMAKE_CURRENT_SOURCE_DIR}/libjit/libjit_conv.cpp ${CMAKE_CURRENT_SOURCE_DIR}/libjit/libjit_matmul.cpp
  WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR})

add_custom_target(CPURuntime
  DEPENDS libjit.bc libjit_conv.bc libjit_matmul.bc
  COMMAND ${LLVM_LINK_BIN} -o ${CMAKE_BINARY_DIR}/libjit.bc libjit.bc libjit_conv.bc libjit_matmul.bc
  WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR})

add_library(CPURuntimeNative
              libjit/libjit.cpp
              libjit/libjit_conv.cpp
              libjit/libjit_matmul.cpp)

add_library(CPUBackend
            AllocationsInfo.cpp
            DebugInfo.cpp
            FunctionSpecializer.cpp
            GlowJIT.cpp
            Pipeline.cpp
            Transforms.cpp
            LLVMIRGen.cpp
            CPUBackend.cpp)

llvm_map_components_to_libnames(LLVM_TARGET_LIBRARIES ${LLVM_TARGETS_TO_BUILD})
target_link_libraries(CPUBackend
                      PRIVATE
                        Base
                        CodeGen
                        Graph
                        IR
                        Quantization
                        LLVMAnalysis
                        LLVMCodeGen
                        LLVMCore
                        LLVMipo
                        LLVMIRReader
                        LLVMInstCombine
                        LLVMMC
                        LLVMScalarOpts
                        LLVMSupport
                        LLVMTarget
                        LLVMTransformUtils
                        LLVMVectorize
                        ${LLVM_TARGET_LIBRARIES}
                        LLVMCore
                        LLVMExecutionEngine
                        LLVMInterpreter
                        LLVMSupport
                        LLVMPasses)
add_dependencies(CPUBackend CPURuntime)
