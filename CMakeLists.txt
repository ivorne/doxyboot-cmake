cmake_minimum_required( VERSION 3.10 )
project( doxyboot-cmake )

include( "cmake/get_target_property_recursive.cmake" )

set_property( GLOBAL PROPERTY DOXYBOOT_SOURCE_DIR_PROP ${CMAKE_CURRENT_SOURCE_DIR} )
set_property( GLOBAL PROPERTY DOXYBOOT_BINARY_DIR_PROP ${CMAKE_CURRENT_BINARY_DIR} )

# Variables used by doxyboot_add_docs:
#    DOXYBOOT_HTML_HEADER - Html file containing <div class="container"></div> with content displayed on top of the page.
#    DOXYBOOT_HTML_FOOTER - Html file containing <div class="container"></div> with content displayed on the bottom of the page.
#    DOXYBOOT_HTML_EXTRA_STYLESHEET - Use instead of DOXYGEN_HTML_EXTRA_STYLESHEET.
#    DOXYBOOT_HTML_EXTRA_FILES - Use instead of DOXYGEN_HTML_EXTRA_FILES.
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
        
        # prepare stylesheet list
        set( EXTRA_STYLESHEETS )
        foreach( FILEPATH ${DOXYBOOT_HTML_EXTRA_STYLESHEET} )
            if( IS_ABSOLUTE "${FILEPATH}" )
                set( EXTRA_STYLESHEETS ${EXTRA_STYLESHEETS} "${FILEPATH}" )
            else()
                set( EXTRA_STYLESHEETS ${EXTRA_STYLESHEETS} "${CMAKE_CURRENT_SOURCE_DIR}/${FILEPATH}" )
            endif()
        endforeach()
        
        # prepare extra files list
        set( EXTRA_FILES )
        foreach( FILEPATH ${DOXYBOOT_HTML_EXTRA_FILES} )
            if( IS_ABSOLUTE "${FILEPATH}" )
                set( EXTRA_FILES ${EXTRA_FILES} "${FILEPATH}" )
            else()
                set( EXTRA_FILES ${EXTRA_FILES} "${CMAKE_CURRENT_SOURCE_DIR}/${FILEPATH}" )
            endif()
        endforeach()
        
        # style - forced
        set( DOXYGEN_HTML_HEADER "${DOXYBOOT_BINARY_DIR}/${target_name}_header.html" )
        set( DOXYGEN_HTML_FOOTER "${DOXYBOOT_BINARY_DIR}/${target_name}_footer.html" )
        set( DOXYGEN_HTML_EXTRA_STYLESHEET "${DOXYBOOT_SOURCE_DIR}/theme/customdoxygen.css" ${EXTRA_STYLESHEETS} )
        set( DOXYGEN_HTML_EXTRA_FILES "${DOXYBOOT_SOURCE_DIR}/theme/doxy-boot.js" ${EXTRA_FILES} )
        set( DOXYGEN_HTML_DYNAMIC_MENUS "NO" )
        set( DOXYGEN_GENERATE_TREEVIEW "NO" )
        set( DOXYGEN_GENERATE_HTML "YES" )
        set( DOXYGEN_HTML_TIMESTAMP "YES" )
        set( DOXYGEN_GENERATE_TAGFILE "${CMAKE_CURRENT_BINARY_DIR}/html/${cpp_target}.tag" )
        
        # bugs - forced
        if( NOT DOXYGEN_SOURCE_TOOLTIPS )
            set( DOXYGEN_SOURCE_TOOLTIPS "NO" )  # does not work, causes errors in console and disrupts bootstrap scripts
        endif()
        
        # options - optional
        #set( DOXYGEN_PROJECT_NUMBER "0.0.0" )
        if( NOT DOXYGEN_JAVADOC_AUTOBRIEF )
            set( DOXYGEN_JAVADOC_AUTOBRIEF "YES" )
        endif()
        
        if( NOT DOXYGEN_BUILTIN_STL_SUPPORT )
            set( DOXYGEN_BUILTIN_STL_SUPPORT "YES" )
        endif()
        
        if( NOT DOXYGEN_SOURCE_BROWSER )
            set( DOXYGEN_SOURCE_BROWSER "YES" )
        endif()
        
        if( NOT DOXYGEN_COLS_IN_ALPHA_INDEX )
            set( DOXYGEN_COLS_IN_ALPHA_INDEX "2" )
        endif()
        
        if( NOT DOXYGEN_CLASS_DIAGRAMS )
            set( DOXYGEN_CLASS_DIAGRAMS "NO" )
        endif()
        
        if( NOT DOXYGEN_CLASS_GRAPH )
            set( DOXYGEN_CLASS_GRAPH "NO" )
        endif()
        
        if( NOT DOXYGEN_COLLABORATION_GRAPH )
            set( DOXYGEN_COLLABORATION_GRAPH "NO" )
        endif()
        
        if( NOT DOXYGEN_INCLUDE_GRAPH )
            set( DOXYGEN_INCLUDE_GRAPH "NO" )
        endif()
        
        if( NOT DOXYGEN_INCLUDED_BY_GRAPH )
            set( DOXYGEN_INCLUDED_BY_GRAPH "NO" )
        endif()
        
        if( NOT DOXYGEN_DOT_IMAGE_FORMAT )
            set( DOXYGEN_DOT_IMAGE_FORMAT "svg" )
        endif()
        
        if( NOT DOXYGEN_EXTRACT_ALL )
            set( DOXYGEN_EXTRACT_ALL "YES" )
        endif()
        
        if( NOT DOXYGEN_SORT_MEMBER_DOCS )
            set( DOXYGEN_SORT_MEMBER_DOCS "NO" )
        endif()
        
        if( NOT DOXYGEN_SHOW_NAMESPACES )
            set( DOXYGEN_SHOW_NAMESPACES "NO" )
        endif()
        
        if( NOT DOXYGEN_EXTERNAL_GROUPS )
            set( DOXYGEN_EXTERNAL_GROUPS "NO" )
        endif()
        
        if( NOT DOXYGEN_EXTERNAL_PAGES )
            set( DOXYGEN_EXTERNAL_PAGES "NO" )
        endif()
        
        # warnings - optional
        #if( NOT DOXYGEN_WARN_IF_UNDOCUMENTED )
        #    set( DOXYGEN_WARN_IF_UNDOCUMENTED "NO" )
        #endif()
        
        # include directories - optional
        if( NOT DOXYGEN_INCLUDE_PATH )
            get_target_property_recursive( DOX_INCLUDE_DIRECTORIES ${cpp_target} INCLUDE_DIRECTORIES )
            set( DOXYGEN_INCLUDE_PATH ${DOX_INCLUDE_DIRECTORIES} )
        endif()
        
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
