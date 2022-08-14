set(GEN_TEXI_FILES_LIST
	version.texi
	parts.texi
	programmers.texi
	programmer_types.texi
	)

set(DOC_TYPES_LIST
	pdf
	dvi
	ps
	html
	)

if("${CMAKE_BUILD_TYPE}" STREQUAL "Debug")
	set(SCRIPT_DEBUG "TRUE")
endif()

# =====================================
# Definitions
# =====================================

set(GEN_TEXI_SCRIPT "gen_texi.sh")
set(GEN_DOC_SCRIPT "gen_doc.sh")

set(DOC_NAME ${PROJECT_NAME})

set(MAIN_EXE ${CMAKE_BINARY_DIR}/src/avrdude)

# Trying to find a program and proceed even if we didn't find it does not make sense for the
# following ones. Prior to 3.18 we will have to check afterwards with if. That's dumb.
# find_program has a nice REQUIRED option since 3.18.
# cmake 3.18. Enforce fix of this as soon as we get there
# -------------------------------------
# find_program(VAR
# 	NAMES foo bar
#	REQUIRED
#	)
# -------------------------------------
if(${CMAKE_MINIMUM_REQUIRED_VERSION} VERSION_GREATER 3.18)
	message(FATAL_ERROR "Fix find_program by setting REQUIRED flag")
endif()
find_program(AWK_BIN NAMES awk gawk)
if(${AWK_BIN} STREQUAL "AWK_BIN-NOTFOUND")
	message(FATAL_ERROR ${AWK_BIN})
endif()

find_program(MAKEINFO_BIN makeinfo)
if(${MAKEINFO_BIN} STREQUAL "MAKEINFO_BIN-NOTFOUND")
	message(FATAL_ERROR ${MAKEINFO_BIN})
endif()


set(MAIN_TEXI "${DOC_NAME}.texi")
set(STYLE_SHEET "${DOC_NAME}.css")
set(COMMENTS "parts_comments.txt")

# =====================================
# Prepare Build Dir
# =====================================

# The use of a copy after configure is ugly but there is no other way prior to
# cmake 3.20. Enforce fix of this as soon as we get there
# -------------------------------------
# configure_file(${SCRIPT}.in ${SCRIPT}
#	FILE_PERMISSIONS OWNER_EXECUTE OWNER_READ
#	@ONLY
#	)
# -------------------------------------
if(${CMAKE_MINIMUM_REQUIRED_VERSION} VERSION_GREATER 3.20)
	message(FATAL_ERROR "Fix configure_file by setting permissions directly and drop file copy")
endif()
configure_file(${GEN_TEXI_SCRIPT}.in tmp/${GEN_TEXI_SCRIPT} @ONLY)
file(COPY ${CMAKE_CURRENT_BINARY_DIR}/tmp/${GEN_TEXI_SCRIPT}
		DESTINATION ${CMAKE_CURRENT_BINARY_DIR}
		FILE_PERMISSIONS
		OWNER_EXECUTE OWNER_READ
	)

configure_file(${GEN_DOC_SCRIPT}.in tmp/${GEN_DOC_SCRIPT} @ONLY)
file(COPY ${CMAKE_CURRENT_BINARY_DIR}/tmp/${GEN_DOC_SCRIPT}
		DESTINATION ${CMAKE_CURRENT_BINARY_DIR}
		FILE_PERMISSIONS
		OWNER_EXECUTE OWNER_READ
	)

file(COPY ${MAIN_TEXI} DESTINATION ${CMAKE_CURRENT_BINARY_DIR})
file(COPY ${STYLE_SHEET} DESTINATION ${CMAKE_CURRENT_BINARY_DIR})
file(COPY ${COMMENTS} DESTINATION ${CMAKE_CURRENT_BINARY_DIR})

# =====================================
# Texi Targets
# =====================================

foreach(TEXI ${GEN_TEXI_FILES_LIST})

	add_custom_target("${TEXI}_target" DEPENDS ${TEXI})

	add_custom_command(OUTPUT ${TEXI}
		COMMAND ${CMAKE_CURRENT_BINARY_DIR}/${GEN_TEXI_SCRIPT} ${TEXI}
		DEPENDS ${MAIN_EXE} ${GEN_TEXI_SCRIPT}.in
		)

endforeach()

# =====================================
# Doc Targets
# =====================================

list(TRANSFORM GEN_TEXI_FILES_LIST APPEND "_target" OUTPUT_VARIABLE GEN_TEXI_FILES_LIST_TARGETS)
foreach(DOC_TYPE ${DOC_TYPES_LIST})

	set(DOC_FILE "${DOC_TYPE}")

	if("${DOC_TYPE}" STREQUAL "html")
		# required for dependency handling. A directory will always be out of date
		set(OUT_FILE "html/Index.html")

		install(DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/${DOC_FILE}
			DESTINATION ${CMAKE_INSTALL_DOCDIR}
			)
	else()
		set(OUT_FILE "${DOC_FILE}")
		string(PREPEND DOC_FILE "${DOC_NAME}.")

		install(FILES ${CMAKE_CURRENT_BINARY_DIR}/${DOC_FILE}
			DESTINATION ${CMAKE_INSTALL_DOCDIR}
			)
	endif()

	add_custom_target("doc_${DOC_TYPE}_target" DEPENDS ${OUT_FILE})

	add_custom_command(OUTPUT ${OUT_FILE}
		COMMAND ${CMAKE_CURRENT_BINARY_DIR}/${GEN_DOC_SCRIPT} ${DOC_TYPE} ${DOC_FILE}
		MAIN_DEPENDENCY ${MAIN_TEXI}
		DEPENDS ${GEN_TEXI_FILES_LIST_TARGETS} ${GEN_TEXI_FILES_LIST} ${GEN_DOC_SCRIPT}.in
		)

endforeach()

# =====================================
# Main Target
# =====================================

list(TRANSFORM DOC_TYPES_LIST PREPEND "doc_")
list(TRANSFORM DOC_TYPES_LIST APPEND "_target")
add_custom_target("doc_target"
	ALL
	DEPENDS ${DOC_TYPES_LIST}
	)
