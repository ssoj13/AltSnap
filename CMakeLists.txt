cmake_minimum_required(VERSION 3.16)
project(AltSnap VERSION 1.65 LANGUAGES C)

# Set C standard
set(CMAKE_C_STANDARD 99)
set(CMAKE_C_STANDARD_REQUIRED ON)

# Configure for Windows
if(WIN32)
    set(CMAKE_SYSTEM_NAME Windows)
    
    # MSVC specific settings
    if(MSVC)
        # Set architecture to x64
        set(CMAKE_GENERATOR_PLATFORM x64)
        
        # MSVC compiler flags for optimization and Windows compatibility
        set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} /Os /GS- /Gy")
        set(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} /MT")
        set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} /DYNAMICBASE /NXCOMPAT")
        
        # Define Windows version and disable some warnings
        add_definitions(-D_WIN32_WINNT=0x0600 -DWINVER=0x0600 -DUNICODE -D_UNICODE)
        add_definitions(-D__USE_MINGW_ANSI_STDIO=0 -DAPP_NAME=L"AltSnap" -DAPP_NAMEA="AltSnap")
        
        # Disable specific MSVC warnings
        set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} /wd4996 /wd4244 /wd4267 /wd4005 /wd4068 /wd4047 /wd4024 /wd4090")
    endif()
endif()

# Include directories
include_directories(${CMAKE_CURRENT_SOURCE_DIR})

# Define source files for AltSnap.exe
set(ALTSNAP_SOURCES
    altsnap.c
)

# Define source files for hooks.dll
set(HOOKS_SOURCES
    hooks.c
)

# Create resource files
if(WIN32)
    # Main application resources
    configure_file(${CMAKE_CURRENT_SOURCE_DIR}/altsnap.rc ${CMAKE_CURRENT_BINARY_DIR}/altsnap.rc COPYONLY)
    configure_file(${CMAKE_CURRENT_SOURCE_DIR}/window.rc ${CMAKE_CURRENT_BINARY_DIR}/window.rc COPYONLY)
    configure_file(${CMAKE_CURRENT_SOURCE_DIR}/AltSnap.exe.manifest ${CMAKE_CURRENT_BINARY_DIR}/AltSnap.exe.manifest COPYONLY)
    
    # Copy media files
    file(COPY ${CMAKE_CURRENT_SOURCE_DIR}/media DESTINATION ${CMAKE_CURRENT_BINARY_DIR})
    
    # Add resource files to executable
    list(APPEND ALTSNAP_SOURCES altsnap.rc)
    
    # Hooks DLL resources
    configure_file(${CMAKE_CURRENT_SOURCE_DIR}/hooks.rc ${CMAKE_CURRENT_BINARY_DIR}/hooks.rc COPYONLY)
    list(APPEND HOOKS_SOURCES hooks.rc)
endif()

# Create the main executable
add_executable(AltSnap WIN32 ${ALTSNAP_SOURCES})

# Create the hooks DLL
add_library(hooks SHARED ${HOOKS_SOURCES})

# Set output names
set_target_properties(AltSnap PROPERTIES OUTPUT_NAME "AltSnap")
set_target_properties(hooks PROPERTIES OUTPUT_NAME "hooks")

# Link libraries for AltSnap.exe
if(WIN32)
    target_link_libraries(AltSnap
        user32
        kernel32
        gdi32
        comctl32
        advapi32
        shell32
    )
    
    # Link libraries for hooks.dll
    target_link_libraries(hooks
        user32
        kernel32
        gdi32
    )
    
    # Set subsystem to Windows for main executable
    set_target_properties(AltSnap PROPERTIES
        LINK_FLAGS "/SUBSYSTEM:WINDOWS /ENTRY:unfuckWinMain /MANIFEST:NO"
    )
    
    # Set DLL entry point
    set_target_properties(hooks PROPERTIES
        LINK_FLAGS "/DEF:${CMAKE_CURRENT_SOURCE_DIR}/hooks.def"
    )
endif()

# Compiler-specific optimizations
if(MSVC)
    # Enable whole program optimization
    set_target_properties(AltSnap PROPERTIES
        INTERPROCEDURAL_OPTIMIZATION TRUE
    )
    set_target_properties(hooks PROPERTIES
        INTERPROCEDURAL_OPTIMIZATION TRUE
    )
endif()

# Install targets
install(TARGETS AltSnap hooks
    RUNTIME DESTINATION bin
    LIBRARY DESTINATION lib
)

# Install language files
install(DIRECTORY Lang/ DESTINATION bin/Lang)

# Install themes
install(DIRECTORY Themes/ DESTINATION bin/Themes)