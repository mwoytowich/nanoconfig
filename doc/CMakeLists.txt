#
#   Copyright (c) 2012-2013 250bpm s.r.o.  All rights reserved.
#   Copyright (c) 2013 GoPivotal, Inc.  All rights reserved.
#
#   Permission is hereby granted, free of charge, to any person obtaining a copy
#   of this software and associated documentation files (the "Software"),
#   to deal in the Software without restriction, including without limitation
#   the rights to use, copy, modify, merge, publish, distribute, sublicense,
#   and/or sell copies of the Software, and to permit persons to whom
#   the Software is furnished to do so, subject to the following conditions:
#
#   The above copyright notice and this permission notice shall be included
#   in all copies or substantial portions of the Software.
#
#   THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
#   IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
#   FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL
#   THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
#   LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING
#   FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS
#   IN THE SOFTWARE.
#

#  On Windows there are no tools to build the documentation.
if (NOT WIN32)

    #  Documentation topics to generate.
    set (topics

        #  Main page.
        nanoconfig.7

        #  Functions.
        nc_configure.3
        nc_close.3
        nc_term.3

        #  Executables.
        nccat.1
        ncdev.1

        )

    #  Generate the rules to build the documentation.
    #  During the process collect the names of resulting documentation files.
    set (docs)
    foreach (topic ${topics})

        #  Generate intermediate docbook file for the manpage.
        add_custom_command (
            DEPENDS
                ${CMAKE_CURRENT_SOURCE_DIR}/${topic}.txt
                ${CMAKE_CURRENT_SOURCE_DIR}/asciidoc.conf
            COMMAND asciidoc -d manpage -b docbook
                -f ${CMAKE_CURRENT_SOURCE_DIR}/asciidoc.conf -o ${topic}.xml
                ${CMAKE_CURRENT_SOURCE_DIR}/${topic}.txt
            OUTPUT ${topic}.xml)

        #  Generate the manpage.
        add_custom_command (
            DEPENDS ${topic}.xml
            COMMAND xmlto -o ${CMAKE_BINARY_DIR} man ${topic}.xml
            OUTPUT ${CMAKE_BINARY_DIR}/${topic})
        set (docs ${docs} ${CMAKE_BINARY_DIR}/${topic})
        get_filename_component (category ${topic} EXT)
        string (REGEX REPLACE "^\\." "" category "${category}")
        install (FILES "${CMAKE_BINARY_DIR}/${topic}"
            DESTINATION share/man/man${category} OPTIONAL)

        #  Generate the HTML page.
        add_custom_command (
            DEPENDS
                ${CMAKE_CURRENT_SOURCE_DIR}/${topic}.txt
                ${CMAKE_CURRENT_SOURCE_DIR}/asciidoc.conf
                ${CMAKE_CURRENT_SOURCE_DIR}/htmldoc.css
            COMMAND asciidoc -d manpage -b xhtml11
                -a stylesheet=${CMAKE_CURRENT_SOURCE_DIR}/htmldoc.css
                -f ${CMAKE_CURRENT_SOURCE_DIR}/asciidoc.conf
                -o ${CMAKE_BINARY_DIR}/${topic}.html
                ${CMAKE_CURRENT_SOURCE_DIR}/${topic}.txt
            OUTPUT ${CMAKE_BINARY_DIR}/${topic}.html)
        set (docs ${docs} ${CMAKE_BINARY_DIR}/${topic}.html)

    endforeach (topic)

    # 'doc' target builds the documentation.
    add_custom_target(doc DEPENDS ${docs})

endif ()

