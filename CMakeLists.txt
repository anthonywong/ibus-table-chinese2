cmake_minimum_required(VERSION 2.4)
CMAKE_POLICY(VERSION 2.4)
CMAKE_POLICY(SET CMP0005 OLD)
####################################################################
# Project specific information
#
PROJECT(ibus-table-chinese)
SET(PRJ_SUMMARY "Chinese input tables for IBus")

SET(PRJ_DESCRIPTION
"ibus-table-chinese is provides the infrastructure for Chinese input methods.
Input tables themselves are in subpackages."
    )

SET(AUTHORS "Yuwei Yu, Caius 'kaio' Chance, Ding-Yi Chen")
SET(MAINTAINER "Ding-Yi Chen <dchen@redhat.com>")
SET(VENDOR "Red Hat, Inc")
SET(LICENSE "GPLv3+")

####################################################################
# Includes
#
SET(CMAKE_MODULE_PATH "/usr/share/cmake/Modules" ${CMAKE_SOURCE_DIR}/Modules)

INCLUDE(ManageVersion)
LOAD_RELEASE_FILE("RELEASE-NOTES.txt")
INCLUDE(UseUninstall)

INCLUDE(CompileEnv)
SET_USUAL_COMPILE_ENVS()

INCLUDE(CMakeVersion)

####################################################################
# Required
#
FIND_PACKAGE(PkgConfig)
PKG_CHECK_MODULES(IBUS_TABLE REQUIRED ibus-table>=1.2)


####################################################################
# Building
#

ADD_SUBDIRECTORY(tables)


####################################################################
# Installing
#

SET(INSTALL_DOCS ${RELEASE_FILE} AUTHORS ChangeLog COPYING README)
STRING_JOIN(PRJ_DOC_LIST " " ${INSTALL_DOCS})

INSTALL(FILES ${INSTALL_DOCS}
    DESTINATION "${PRJ_DOC_DIR}")

####################################################################
# Packing
#
SET(MAINTAINER_SETTING "MAINTAINER_SETTING_NO_PACK")
INCLUDE(PackSource)
INCLUDE(PackRPM)
INCLUDE(UseFedpkg)

SET(PRJ_IGNORE_FILES_COMMON
    "\\\\.spec$" "messages.po$" "\\\\.orig$" "\\\\.db$"
    "\\\\.utf8$" "\\\\.tmp$"
    "array30.txt" "array30-big.txt" "xinhua.txt" "yong.txt"
)

SET(PACK_SOURCE_IGNORE_FILES ${PACK_SOURCE_IGNORE_FILES}
    ${PRJ_IGNORE_FILES_COMMON} "/bin/")

PACK_SOURCE(PACK_SOURCE_FILE_NAME "${RPM_BUILD_SOURCES}")
PACK_RPM(PRJ_SRPM_FILE_NAME "${RPM_BUILD_SPECS}/project.spec.in" "${PACK_SOURCE_FILE_NAME}")

# If mock exists, then add mock related targets.
FIND_PROGRAM(MOCK_CMD mock)
IF(NOT MOCK_CMD STREQUAL "MOCK_CMD-NOTFOUND")
    USE_MOCK("${RPM_BUILD_SPECS}/project.spec.in")
ENDIF(NOT MOCK_CMD STREQUAL "MOCK_CMD-NOTFOUND")

IF(EXISTS ${MAINTAINER_SETTING})
    USE_FEDPKG("${RPM_BUILD_SRPMS}/${PRJ_SRPM_FILE_NAME}")
    USE_BODHI()
ENDIF(EXISTS ${MAINTAINER_SETTING})

####################################################################
# Hosting
#
INCLUDE(UseHostingService)
USE_HOSTING_SERVICE_READ_SETTING_FILE("${MAINTAINER_SETTING}"
    "${RPM_BUILD_SOURCES}/${PACK_SOURCE_FILE_NAME}")


####################################################################
# Test Suites.
#
#ENABLE_TESTING()
#ADD_TEST("Test" test_cmd1)

