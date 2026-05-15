# Builds a minimal static `faad2` lib (AAC-LC decoder) for emscripten.
#
# Vendoring expected: place the upstream faad2 sources at
#   lib/codec/3rdparty/faad2/libfaad/    (all *.c / *.h)
#   lib/codec/3rdparty/faad2/include/    (neaacdec.h — knik0/upstream layout)
# Older forks keep neaacdec.h inside libfaad/ instead; both layouts work.
#
# Tested against knik0/faad2 (https://github.com/knik0/faad2). Other forks
# work too as long as the libfaad/ source layout is unchanged.

file(GLOB FAAD2_SOURCES CONFIGURE_DEPENDS
        "${CMAKE_CURRENT_SOURCE_DIR}/libfaad/*.c")

add_library(faad2 STATIC ${FAAD2_SOURCES})

target_include_directories(faad2 PUBLIC
        "${CMAKE_CURRENT_SOURCE_DIR}/libfaad")

if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/include/neaacdec.h")
    target_include_directories(faad2 PUBLIC
            "${CMAKE_CURRENT_SOURCE_DIR}/include")
elseif(NOT EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/libfaad/neaacdec.h")
    message(FATAL_ERROR
        "faad2: neaacdec.h not found. Copy it from upstream into either "
        "${CMAKE_CURRENT_SOURCE_DIR}/include/ or "
        "${CMAKE_CURRENT_SOURCE_DIR}/libfaad/. See VENDORING.md.")
endif()

# Skip autoconf-generated config.h; faad2's common.h has sane defaults
# for a hosted floating-point build (no FIXED_POINT, MAIN/LTP/LD decoders on,
# error resilience on). We still have to flag which platform headers exist —
# otherwise common.h falls back to its own uint32_t typedef and collides with
# emscripten's <stdint.h>. HAVE_LRINTF picks the libc fast path for lrintf().
target_compile_definitions(faad2 PRIVATE
        HAVE_STDINT_H
        HAVE_INTTYPES_H
        HAVE_SYS_TYPES_H
        HAVE_SYS_STAT_H
        HAVE_STRING_H
        HAVE_UNISTD_H
        HAVE_LRINTF
        PACKAGE_VERSION="2.11.1")

# faad2 emits a flood of warnings; mute them so they don't drown out the rest.
target_compile_options(faad2 PRIVATE
        -Wno-everything)
