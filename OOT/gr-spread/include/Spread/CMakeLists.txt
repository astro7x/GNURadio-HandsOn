# Copyright 2011,2012 Free Software Foundation, Inc.
#
# This file is part of GNU Radio
#
# GNU Radio is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation; either version 3, or (at your option)
# any later version.
#
# GNU Radio is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with GNU Radio; see the file COPYING.  If not, write to
# the Free Software Foundation, Inc., 51 Franklin Street,
# Boston, MA 02110-1301, USA.

include(GrPython)

file(WRITE ${CMAKE_CURRENT_BINARY_DIR}/generate_helper.py "
#!${PYTHON_EXECUTABLE}

import sys, os, re
sys.path.append('${GR_RUNTIME_PYTHONPATH}')
os.environ['srcdir'] = '${CMAKE_CURRENT_SOURCE_DIR}'
os.chdir('${CMAKE_CURRENT_BINARY_DIR}')

if __name__ == '__main__':
    sys.path.append('${CMAKE_CURRENT_SOURCE_DIR}/../../python')
    import build_utils
    root, inp = sys.argv[1:3]
    for sig in sys.argv[3:]:
        name = re.sub ('X+', sig, root)
        d = build_utils.standard_dict(name, sig, 'Spread')
        build_utils.expand_template(d, inp)

")

macro(expand_h root)
  #make a list of all the generated files
  unset(expanded_files_h)
  foreach(sig ${ARGN})
    string(REGEX REPLACE "X+" ${sig} name ${root})
    list(APPEND expanded_files_h ${CMAKE_CURRENT_BINARY_DIR}/${name}.h)
  endforeach(sig)

  #create a command to generate the files
  add_custom_command(
    OUTPUT ${expanded_files_h}
    DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/${root}.h.t
    COMMAND ${PYTHON_EXECUTABLE} ${PYTHON_DASH_B}
    ${CMAKE_CURRENT_BINARY_DIR}/generate_helper.py
    ${root} ${root}.h.t ${ARGN}
    )

  #install rules for the generated h files
  list(APPEND generated_includes ${expanded_files_h})
endmacro(expand_h)

expand_h(preamble_XX bb cc) 

add_custom_target(Spread_generated_includes DEPENDS
    ${generated_includes}
)

########################################################################
# Install public header files
########################################################################
install(FILES
    ${generated_includes}
    api.h
    synthesizer.h
    framer.h
    deframer.h
    sync.h
    cpfsk_mod.h
    cpfsk_demod.h
    msg_source.h
    rx_synthesizer.h
    ds_spreader.h
    ds_despreader.h
    rx_sync.h DESTINATION include/Spread
)
