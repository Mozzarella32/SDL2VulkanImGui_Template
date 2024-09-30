cmake_minimum_required(VERSION 3.5)
project(ImGuiVulkanSDL2_Project)

include(FetchContent)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Vulkan finden
find_package(Vulkan REQUIRED)

# SDL2 statische Bibliothek herunterladen
message(STATUS "Fetching SDL2")
FetchContent_Declare(
    SDL2
    GIT_REPOSITORY https://github.com/libsdl-org/SDL.git
    GIT_TAG release-2.28.2  # Verwende die neueste stabile Version
)

# Setze Optionen für SDL2
set(SDL_TESTS OFF FORCE)
set(SDL_EXAMPLES OFF FORCE)
set(SDL_STATIC ON FORCE)
set(SDL_SHARED OFF FORCE)

FetchContent_MakeAvailable(SDL2)

# Dear ImGui (Docking Branch) herunterladen
message(STATUS "Fetching ImGui docking")
FetchContent_Declare(
    ImGui
    GIT_REPOSITORY https://github.com/ocornut/imgui.git
    GIT_TAG docking  # Verwende den Docking-Branch
)
FetchContent_MakeAvailable(ImGui)

# Directorys
set(SRC_DIR "${PROJECT_SOURCE_DIR}/source")
set(INCLUDE_DIR "${PROJECT_SOURCE_DIR}/include")
set(BIN_DIR "${PROJECT_SOURCE_DIR}/bin")

# ImGui Quell- und Backend-Dateien definieren
set(IMGUI_SOURCES
    ${imgui_SOURCE_DIR}/imgui.cpp
    ${imgui_SOURCE_DIR}/imgui_demo.cpp
    ${imgui_SOURCE_DIR}/imgui_draw.cpp
    ${imgui_SOURCE_DIR}/imgui_tables.cpp
    ${imgui_SOURCE_DIR}/imgui_widgets.cpp
    ${imgui_SOURCE_DIR}/backends/imgui_impl_sdl2.cpp
    ${imgui_SOURCE_DIR}/backends/imgui_impl_vulkan.cpp
)

set(PCH_HEADERS 
    #"${INCLUDE_DIR}/pch.hpp"
)

# Glob alle Quell- und Header-Dateien
file(GLOB_RECURSE MY_SOURCE CONFIGURE_DEPENDS "${SRC_DIR}/*")
file(GLOB_RECURSE MY_HEADERS CONFIGURE_DEPENDS "${INCLUDE_DIR}/*")

source_group("Source Files" FILES ${MY_SOURCE})
source_group("Header Files" FILES ${MY_HEADERS})

# PCH-Dateien aus den Quellen und Headern entfernen
list(REMOVE_ITEM MY_SOURCE ${PCH_HEADERS})
list(REMOVE_ITEM MY_HEADERS ${PCH_HEADERS})

# Füge ImGui-Quellen zur MY_SOURCE-Liste hinzu
#list(APPEND MY_SOURCE ${IMGUI_SOURCES})

add_executable(${PROJECT_NAME} ${MY_SOURCE} ${IMGUI_SOURCES} ${MY_HEADERS})

# Präkompilierte Header konfigurieren (wenn vorhanden)
target_precompile_headers(${PROJECT_NAME} PRIVATE ${PCH_HEADERS})

# Setze das Output-Verzeichnis für jede Konfiguration
set_target_properties(${PROJECT_NAME} PROPERTIES
    RUNTIME_OUTPUT_DIRECTORY "${BIN_DIR}"
    RUNTIME_OUTPUT_DIRECTORY_DEBUG "${BIN_DIR}"
    RUNTIME_OUTPUT_DIRECTORY_RELEASE "${BIN_DIR}"
    RUNTIME_OUTPUT_DIRECTORY_RELWITHDEBINFO "${BIN_DIR}"
    RUNTIME_OUTPUT_DIRECTORY_MINSIZEREL "${BIN_DIR}"
)

# Output-Namen für Debug/Release Konfigurationen setzen
set_target_properties(${PROJECT_NAME} PROPERTIES 
    OUTPUT_NAME_DEBUG "${PROJECT_NAME}_d"
    OUTPUT_NAME_RELEASE "${PROJECT_NAME}"
    OUTPUT_NAME_RELWITHDEBINFO "${PROJECT_NAME}"
    OUTPUT_NAME_MINSIZEREL "${PROJECT_NAME}"
)

# Subsystem für Windows
if(WIN32)
    set_target_properties(${PROJECT_NAME} PROPERTIES WIN32_EXECUTABLE TRUE)
endif()

# Include-Verzeichnisse
target_include_directories(${PROJECT_NAME} 
    PUBLIC ${INCLUDE_DIR}
    PUBLIC ${imgui_SOURCE_DIR}
    PUBLIC ${imgui_SOURCE_DIR}/backends
)

# SDL2 und Vulkan verlinken
target_link_libraries(${PROJECT_NAME}
    PRIVATE SDL2::SDL2-static
    PRIVATE Vulkan::Vulkan
)

# Vulkan-spezifische Definition
target_compile_definitions(${PROJECT_NAME} PRIVATE IMGUI_IMPL_VULKAN)
