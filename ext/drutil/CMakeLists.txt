# **********************************************************
# Copyright (c) 2010-2014 Google, Inc.    All rights reserved.
# **********************************************************

# drutil: DynamoRIO Function Wrapping and Replacing Extension
# Derived from Dr. Memory: the memory debugger
#
# This library is free software; you can redistribute it and/or
# modify it under the terms of the GNU Lesser General Public
# License as published by the Free Software Foundation;
# version 2.1 of the License, and no later version.
#
# This library is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU
# Library General Public License for more details.
#
# You should have received a copy of the GNU Lesser General Public
# License along with this library; if not, write to the Free Software
# Foundation, Inc., 51 Franklin Street, Fifth Floor, Boston, MA 02110-1301 USA.

cmake_minimum_required(VERSION 2.6)

# DynamoRIO Function Wrapping and Replacing Extension

# We do not need libc, and we avoid a 10x size increase in both our
# dll and pdb (plus we avoid stressing private library isolation) by
# not using it (i#714).
set(DynamoRIO_USE_LIBC OFF)

set(srcs
  drutil.c
  # add more here
  )

# i#1491#2: VS generators fail if static lib has resources
set(srcs_static ${srcs})

if (WIN32)
  set(srcs ${srcs} ${PROJECT_SOURCE_DIR}/core/win32/resources.rc)
endif ()

add_library(drutil ${libtype} ${srcs})
# while private loader means preferred base is not required, more efficient
# to avoid rebase so we avoid conflict w/ client and other exts
set(PREFERRED_BASE 0x75000000)
configure_extension(drutil)
use_DynamoRIO_extension(drutil drmgr)

# Since LGPL, most users will want this as a shared library.
# A shared library is also required if multiple separate components all want to
# use this same extension.
# We do provide the static library for those who are ok w/ the license and who
# are building an end tool.
if (NOT STATIC_LIBRARY)
  add_library(drutil_static STATIC ${srcs_static})
  configure_extension(drutil_static)
  use_DynamoRIO_extension(drutil_static drmgr_static)
endif ()

DR_install(FILES drutil.h DESTINATION ${INSTALL_EXT_INCLUDE})
