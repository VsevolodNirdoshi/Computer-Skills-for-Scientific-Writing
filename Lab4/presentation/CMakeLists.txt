cmake_minimum_required(VERSION 3.10)
project(PresentationDocuments)

# Find pandoc
find_program(PANDOC_EXECUTABLE pandoc)

if(PANDOC_EXECUTABLE)
    message(STATUS "Found pandoc: ${PANDOC_EXECUTABLE}")
    
    # Find all markdown files in the current directory
    file(GLOB MD_FILES "*.md")
    
    # Remove this CMakeLists.txt from the file list if it exists
    list(FILTER MD_FILES EXCLUDE REGEX "CMakeLists.txt")
    
    if(NOT MD_FILES)
        message(WARNING "No markdown files found in directory!")
        return()
    endif()
    
    message(STATUS "Found markdown files: ${MD_FILES}")
    
    # Create targets for each markdown file
    foreach(MD_FILE ${MD_FILES})
        # Get the base filename without extension
        get_filename_component(BASE_NAME ${MD_FILE} NAME_WE)
        
        message(STATUS "Creating presentation targets for: ${BASE_NAME}")
        
        # PDF target (Beamer presentation)
        add_custom_command(
            OUTPUT ${CMAKE_CURRENT_SOURCE_DIR}/${BASE_NAME}.pdf
            COMMAND ${PANDOC_EXECUTABLE} 
                    ${MD_FILE}
                    --pdf-engine=lualatex
                    --citeproc
                    --number-sections
                    -t beamer 
                    --slide-level=2
                    -o ${BASE_NAME}.pdf
            DEPENDS ${MD_FILE}
            COMMENT "Generating PDF presentation: ${BASE_NAME}.pdf"
            WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
        )
        
        # HTML target (Reveal.js presentation)
        add_custom_command(
            OUTPUT ${CMAKE_CURRENT_SOURCE_DIR}/${BASE_NAME}.html
            COMMAND ${PANDOC_EXECUTABLE} 
                    ${MD_FILE}
                    --embed-resources 
                    --standalone 
                    -t revealjs 
                    -V theme=beige
                    -o ${BASE_NAME}.html
            DEPENDS ${MD_FILE}
            COMMENT "Generating HTML presentation: ${BASE_NAME}.html"
            WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
        )
        
        # Add to the list of output files
        list(APPEND PDF_OUTPUTS ${CMAKE_CURRENT_SOURCE_DIR}/${BASE_NAME}.pdf)
        list(APPEND HTML_OUTPUTS ${CMAKE_CURRENT_SOURCE_DIR}/${BASE_NAME}.html)
        list(APPEND ALL_OUTPUTS ${CMAKE_CURRENT_SOURCE_DIR}/${BASE_NAME}.pdf ${CMAKE_CURRENT_SOURCE_DIR}/${BASE_NAME}.html)
        
    endforeach()
    
    # Main target that builds all documents
    add_custom_target(build_all ALL 
        DEPENDS ${ALL_OUTPUTS}
        COMMENT "Building all presentation documents"
    )
    
    # Individual targets for specific formats
    add_custom_target(build_pdf
        DEPENDS ${PDF_OUTPUTS}
        COMMENT "Building all PDF presentations"
    )
    
    add_custom_target(build_html
        DEPENDS ${HTML_OUTPUTS}
        COMMENT "Building all HTML presentations"
    )
    
    # Per-file targets
    foreach(MD_FILE ${MD_FILES})
        get_filename_component(BASE_NAME ${MD_FILE} NAME_WE)
        
        add_custom_target(${BASE_NAME}_pdf
            DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/${BASE_NAME}.pdf
            COMMENT "Building ${BASE_NAME}.pdf (Beamer)"
        )
        
        add_custom_target(${BASE_NAME}_html
            DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/${BASE_NAME}.html
            COMMENT "Building ${BASE_NAME}.html (Reveal.js)"
        )
        
    endforeach()
    
else()
    message(FATAL_ERROR "Pandoc not found! Please install pandoc.")
endif()

# Clean target
add_custom_target(clean_all
    COMMAND ${CMAKE_COMMAND} -E remove ${ALL_OUTPUTS}
    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
    COMMENT "Cleaning all generated presentations"
)

# Show available targets
message(STATUS "Available presentation targets:")
message(STATUS "  cmake --build build --target build_all    # Build all presentations")
message(STATUS "  cmake --build build --target build_pdf    # Build all PDF (Beamer)")  
message(STATUS "  cmake --build build --target build_html   # Build all HTML (Reveal.js)")
foreach(MD_FILE ${MD_FILES})
    get_filename_component(BASE_NAME ${MD_FILE} NAME_WE)
    message(STATUS "  cmake --build build --target ${BASE_NAME}_pdf   # Build ${BASE_NAME}.pdf")
    message(STATUS "  cmake --build build --target ${BASE_NAME}_html  # Build ${BASE_NAME}.html")
endforeach()
message(STATUS "  cmake --build build --target clean_all    # Clean all generated files")