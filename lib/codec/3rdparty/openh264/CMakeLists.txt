file(GLOB OPENH264_COMMON_SOURCES CONFIGURE_DEPENDS
        "${CMAKE_CURRENT_SOURCE_DIR}/codec/common/src/*.cpp")

file(GLOB OPENH264_DECODER_SOURCES CONFIGURE_DEPENDS
        "${CMAKE_CURRENT_SOURCE_DIR}/codec/decoder/core/src/*.cpp"
        "${CMAKE_CURRENT_SOURCE_DIR}/codec/decoder/plus/src/*.cpp")

add_library(openh264 STATIC
        ${OPENH264_COMMON_SOURCES}
        ${OPENH264_DECODER_SOURCES})

target_include_directories(openh264 PUBLIC
        "${CMAKE_CURRENT_SOURCE_DIR}/codec/api/svc"
        "${CMAKE_CURRENT_SOURCE_DIR}/codec/common/inc"
        "${CMAKE_CURRENT_SOURCE_DIR}/codec/decoder/core/inc"
        "${CMAKE_CURRENT_SOURCE_DIR}/codec/decoder/plus/inc")

# Decoder-only build. No X86_ASM / HAVE_NEON: falls back to the portable C++
# paths in cpu.cpp/mc.cpp/etc., which is what we want under emscripten.
target_compile_definitions(openh264 PRIVATE
        HAVE_STDINT_H)

target_compile_options(openh264 PRIVATE
        -Wno-everything)
