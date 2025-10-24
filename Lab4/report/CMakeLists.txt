cmake_minimum_required(VERSION 3.10)
project(MarkdownDocuments)

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
        
        message(STATUS "Creating targets for: ${BASE_NAME}")
        
        # PDF target
        add_custom_command(
            OUTPUT ${CMAKE_CURRENT_SOURCE_DIR}/${BASE_NAME}.pdf
            COMMAND ${PANDOC_EXECUTABLE} 
                    ${MD_FILE}
                    --pdf-engine=lualatex
                    --citeproc
                    -o ${BASE_NAME}.pdf
            DEPENDS ${MD_FILE} ${CMAKE_CURRENT_SOURCE_DIR}/cite.bib
            COMMENT "Generating PDF: ${BASE_NAME}.pdf"
            WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
        )
        
        # DOCX target  
        add_custom_command(
            OUTPUT ${CMAKE_CURRENT_SOURCE_DIR}/${BASE_NAME}.docx
            COMMAND ${PANDOC_EXECUTABLE} 
                    ${MD_FILE}
                    --citeproc
                    -o ${BASE_NAME}.docx
            DEPENDS ${MD_FILE} ${CMAKE_CURRENT_SOURCE_DIR}/cite.bib
            COMMENT "Generating DOCX: ${BASE_NAME}.docx"
            WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
        )
        
        # Add to the list of output files
        list(APPEND PDF_OUTPUTS ${CMAKE_CURRENT_SOURCE_DIR}/${BASE_NAME}.pdf)
        list(APPEND DOCX_OUTPUTS ${CMAKE_CURRENT_SOURCE_DIR}/${BASE_NAME}.docx)
        list(APPEND ALL_OUTPUTS ${CMAKE_CURRENT_SOURCE_DIR}/${BASE_NAME}.pdf ${CMAKE_CURRENT_SOURCE_DIR}/${BASE_NAME}.docx)
        
    endforeach()
    
    # Main target that builds all documents
    add_custom_target(build_all ALL 
        DEPENDS ${ALL_OUTPUTS}
        COMMENT "Building all markdown documents"
    )
    
    # Individual targets for specific file types
    add_custom_target(build_pdf
        DEPENDS ${PDF_OUTPUTS}
        COMMENT "Building all PDF documents"
    )
    
    add_custom_target(build_docx
        DEPENDS ${DOCX_OUTPUTS}
        COMMENT "Building all DOCX documents"
    )
    
    # Per-file targets
    foreach(MD_FILE ${MD_FILES})
        get_filename_component(BASE_NAME ${MD_FILE} NAME_WE)
        
        add_custom_target(${BASE_NAME}_pdf
            DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/${BASE_NAME}.pdf
            COMMENT "Building ${BASE_NAME}.pdf"
        )
        
        add_custom_target(${BASE_NAME}_docx
            DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/${BASE_NAME}.docx
            COMMENT "Building ${BASE_NAME}.docx"
        )
        
    endforeach()
    
else()
    message(FATAL_ERROR "Pandoc not found! Please install pandoc.")
endif()

# Clean target
add_custom_target(clean_all
    COMMAND ${CMAKE_COMMAND} -E remove ${ALL_OUTPUTS}
    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
    COMMENT "Cleaning all generated documents"
)

# Show available targets
message(STATUS "Available targets:")
message(STATUS "  cmake --build build --target build_all    # Build all documents")
message(STATUS "  cmake --build build --target build_pdf    # Build all PDFs")  
message(STATUS "  cmake --build build --target build_docx   # Build all DOCXs")
foreach(MD_FILE ${MD_FILES})
    get_filename_component(BASE_NAME ${MD_FILE} NAME_WE)
    message(STATUS "  cmake --build build --target ${BASE_NAME}_pdf   # Build ${BASE_NAME}.pdf")
    message(STATUS "  cmake --build build --target ${BASE_NAME}_docx  # Build ${BASE_NAME}.docx")
endforeach()
message(STATUS "  cmake --build build --target clean_all    # Clean all generated files")