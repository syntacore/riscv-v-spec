cmake_minimum_required(VERSION 3.6)

project(v-spec VERSION 1.0 LANGUAGES NONE)

find_package(Git)

if(Git_FOUND)
    execute_process(
        COMMAND ${GIT_EXECUTABLE} -C ${CMAKE_CURRENT_SOURCE_DIR} describe --tag --dirty
        WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
        OUTPUT_VARIABLE GIT_DESCRIBE
        OUTPUT_STRIP_TRAILING_WHITESPACE
    )
    message(STATUS "${GIT_DESCRIBE}")
else()
    set(GIT_DESCRIBE "Unknown version" CACHE STRING "User-defined version string")
endif()

FIND_PROGRAM(ASCIIDOCTOR_EXECUTABLE "asciidoctor")

IF (NOT ASCIIDOCTOR_EXECUTABLE)
  MESSAGE(FATAL_ERROR "Could not find asciidoctor")
ENDIF (NOT ASCIIDOCTOR_EXECUTABLE)

FIND_PROGRAM(ASCIIDOCTORPDF_EXECUTABLE "asciidoctor-pdf")

IF (NOT ASCIIDOCTORPDF_EXECUTABLE)
  MESSAGE(FATAL_ERROR "Could not find asciidoctor-pdf")
ENDIF (NOT ASCIIDOCTORPDF_EXECUTABLE)

set(ADOC_LIBS
    # asciidoctor-diagram
    # asciimath
    # asciidoctor-plantuml
)

set(ADOC_ATTR
    # allow-uri-read
    # asciidoctor-mathematical
    # stem=latexmath
    # mathematical-format=svg
    pdf-page-size=A4
    VERSION=${GIT_DESCRIBE}
)

set(ADOCFLAGS --safe -D ${CMAKE_CURRENT_BINARY_DIR})

foreach(attr ${ADOC_ATTR})
    list(APPEND ADOCFLAGS --attribute ${attr})
endforeach()

foreach(lib ${ADOC_LIBS})
    list(APPEND ADOCFLAGS --require ${lib})
endforeach()

file(GLOB ADOC_SRC LIST_DIRECTORIES false *.adoc)

macro(ADOC ext backend exe)
    add_custom_command(
        OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}.${ext}
        COMMAND ${exe} ${ADOCFLAGS} --backend ${backend} -o ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}.${ext} ${CMAKE_CURRENT_SOURCE_DIR}/${PROJECT_NAME}.adoc
        DEPENDS ${ADOC_SRC}
    )
    add_custom_target(${PROJECT_NAME}-${backend} ALL DEPENDS ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}.${ext})
endmacro()

ADOC(pdf pdf ${ASCIIDOCTORPDF_EXECUTABLE})
ADOC(html html ${ASCIIDOCTOR_EXECUTABLE})
ADOC(xml docbook ${ASCIIDOCTOR_EXECUTABLE})
set_target_properties(${PROJECT_NAME}-docbook PROPERTIES EXCLUDE_FROM_ALL 1)
