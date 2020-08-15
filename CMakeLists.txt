cmake_minimum_required( VERSION 3.10 )
project( doxyboot-cmake )

include( "cmake/get_target_property_recursive.cmake" )

set_property( GLOBAL PROPERTY DOXYBOOT_SOURCE_DIR_PROP ${CMAKE_CURRENT_SOURCE_DIR} )
set_property( GLOBAL PROPERTY DOXYBOOT_BINARY_DIR_PROP ${CMAKE_CURRENT_BINARY_DIR} )

# Variables used by doxyboot_add_docs:
#    DOXYBOOT_HTML_HEADER - Html file containing <div class="container"></div> with content displayed on top of the page.
#    DOXYBOOT_HTML_FOOTER - Html file containing <div class="container"></div> with content displayed on the bottom of the page.
#
function( doxyboot_add_docs target_name cpp_target src_dir )
    find_package( Doxygen 1.8.17 )
    
    if( DOXYGEN_FOUND )

        message( STATUS "Adding doxyboot target ${target_name}" )
        get_property( DOXYBOOT_SOURCE_DIR GLOBAL PROPERTY DOXYBOOT_SOURCE_DIR_PROP )
        get_property( DOXYBOOT_BINARY_DIR GLOBAL PROPERTY DOXYBOOT_BINARY_DIR_PROP )
        
        # prepare header file
        file( READ "${DOXYBOOT_HTML_HEADER}" STR_CUSTOM_HEADER )
        file( READ "${DOXYBOOT_SOURCE_DIR}/theme/header.html" STR_HEADER_TEMPLATE )
        string( REPLACE "%{doxyboot_header}" "${STR_CUSTOM_HEADER}" STR_HEADER_RESULT "${STR_HEADER_TEMPLATE}" )
        file( WRITE "${DOXYBOOT_BINARY_DIR}/${target_name}_header.html" ${STR_HEADER_RESULT} )
        
        # prepare footer file
        file( READ "${DOXYBOOT_HTML_FOOTER}" STR_CUSTOM_FOOTER )
        file( READ "${DOXYBOOT_SOURCE_DIR}/theme/footer.html" STR_FOOTER_TEMPLATE )
        string( REPLACE "%{doxyboot_footer}" "${STR_CUSTOM_FOOTER}" STR_FOOTER_RESULT "${STR_FOOTER_TEMPLATE}" )
        file( WRITE "${DOXYBOOT_BINARY_DIR}/${target_name}_footer.html" ${STR_FOOTER_RESULT} )
        
        # style
        set( DOXYGEN_HTML_HEADER "${DOXYBOOT_BINARY_DIR}/${target_name}_header.html" )
        set( DOXYGEN_HTML_FOOTER "${DOXYBOOT_BINARY_DIR}/${target_name}_footer.html" )
        set( DOXYGEN_HTML_EXTRA_STYLESHEET "${DOXYBOOT_SOURCE_DIR}/theme/customdoxygen.css" )
        set( DOXYGEN_HTML_EXTRA_FILES "${DOXYBOOT_SOURCE_DIR}/theme/doxy-boot.js" )
        set( DOXYGEN_HTML_DYNAMIC_MENUS "NO" )
        set( DOXYGEN_GENERATE_TREEVIEW "NO" )
        set( DOXYGEN_GENERATE_HTML "YES" )
        set( DOXYGEN_HTML_TIMESTAMP "YES" )
        
        # options
        #set( DOXYGEN_CLANG_ASSISTED_PARSING "YES" )
        set( DOXYGEN_PROJECT_NUMBER "0.0.0" )
        set( DOXYGEN_BUILTIN_STL_SUPPORT "YES" )
        set( DOXYGEN_SOURCE_BROWSER "YES" )
        set( DOXYGEN_COLS_IN_ALPHA_INDEX "2" )
        set( DOXYGEN_CLASS_DIAGRAMS "NO" )
        set( DOXYGEN_CLASS_GRAPH "NO" )
        set( DOXYGEN_COLLABORATION_GRAPH "NO" )
        set( DOXYGEN_INCLUDE_GRAPH "NO" )
        set( DOXYGEN_INCLUDED_BY_GRAPH "NO" )
        set( DOXYGEN_DOT_IMAGE_FORMAT "svg" )
        set( DOXYGEN_EXTRACT_ALL "YES" )
        set( DOXYGEN_SORT_MEMBER_DOCS "NO" )
        set( DOXYGEN_GENERATE_TAGFILE "${CMAKE_CURRENT_BINARY_DIR}/html/${cpp_target}.tag" )
        set( DOXYGEN_SHOW_NAMESPACES "NO" )
        set( DOXYGEN_EXTERNAL_GROUPS "NO" )
        set( DOXYGEN_EXTERNAL_PAGES "NO" )
        
        # bugs
        set( DOXYGEN_SOURCE_TOOLTIPS "NO" )  # does not work, causes errors in console and disrupts bootstrap scripts
        
        # warnings
        set( DOXYGEN_WARN_IF_UNDOCUMENTED "NO" )
        
        # include directories
        get_target_property_recursive( DOX_INCLUDE_DIRECTORIES ${cpp_target} INCLUDE_DIRECTORIES )
        set( DOXYGEN_INCLUDE_PATH ${DOX_INCLUDE_DIRECTORIES} )
        
        # src_dir preprocess
        if( IS_ABSOLUTE "${src_dir}" )
            set( SOURCE_DIR "${src_dir}" )
        else()
            set( SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/${src_dir}" )
        endif()
        
        # documentation target
        doxygen_add_docs( ${target_name} "." WORKING_DIRECTORY ${SOURCE_DIR} )
        
    else()
    
        message( WARNING "Skipping doxyboot target ${target_name} - Doxygen >=1.8.17 was not found." )
        
    endif()
endfunction()
