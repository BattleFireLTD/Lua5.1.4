CMAKE_MINIMUM_REQUIRED(VERSION 2.6)
set(PROJECT_NAME "Lua5.1.4")
Project(${PROJECT_NAME})

function(GroupDirectories dirPrefix groupPrefix targetDir luaLibSources luaCompilerSources)
	get_filename_component(currentDirName ${targetDir} NAME)
	string(CONCAT currentDirFullName "${dirPrefix}${currentDirName}/")
	string(CONCAT currentDirGroupName "${groupPrefix}${currentDirName}")
	file(GLOB allFiles "${targetDir}/*")
	file(GLOB allCFiles "${targetDir}/*.c")
	file(GLOB allHFiles "${targetDir}/*.h*")
	file(GLOB allCXXFiles "${targetDir}/*.c*")
	
	list(LENGTH allFiles  argv_len)
	list(LENGTH allCFiles lenC)
	list(LENGTH allCXXFiles lenCXX)
	list(LENGTH allHFiles lenH)
	
	set(allUsefulFileCount 0)
	math(EXPR allUsefulFileCount "${lenC}+${lenCXX}+${lenH}")
	#message(STATUS "useful file count ${allUsefulFileCount}")
	if(0 LESS allUsefulFileCount)
		set(mGroups ${allCFiles} ${allCXXFiles} ${allHFiles})
		source_group(${currentDirGroupName} FILES ${mGroups})
		message(STATUS "${currentDirGroupName} c:${lenC} cxx:${lenCXX} h:${lenH} total:${argv_len}")
	endif()
	
	list(REMOVE_ITEM mGroups ${targetDir}/lua.c)
	set(${luaCompilerSources} ${mGroups} PARENT_SCOPE)
	
	list(REMOVE_ITEM mGroups ${targetDir}/luac.c;)
	set(${luaLibSources} ${mGroups} PARENT_SCOPE)
endfunction()

if(WIN32)
	message(STATUS "PROJECT_SOURCE_DIR ${PROJECT_SOURCE_DIR}")
	message(STATUS "CMAKE_CURRENT_SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}")
	include_directories(${PROJECT_SOURCE_DIR}/src)
	GroupDirectories("" "" ${CMAKE_CURRENT_SOURCE_DIR}/src luaLibSources luaCompilerSources)
	
	message(STATUS "${luaLibSources}")
	
	set(LuaLibModule ${luaLibSources})
	set(LuaCompilerModule ${luaCompilerSources})
	
	add_definitions(-DUNICODE -D_UNICODE -D_SCL_SECURE_NO_WARNINGS -D_CRT_SECURE_NO_WARNINGS)
	set(LIBRARY_OUTPUT_PATH ${PROJECT_BINARY_DIR}/lib)

	set(CMAKE_CXX_FLAGS_RELEASE "/MT /O2 /Ob2 /DNDEBUG")
	set(CMAKE_CXX_FLAGS_DEBUG "/MTd /Zi /Ob0 /Od /RTC1")
	set(CMAKE_CXX_FLAGS "/DWIN32 /D_WINDOWS /W3 /GR /EHsc")
	set(CMAKE_C_FLAGS_RELEASE "/MT /O2 /Ob2 /DNDEBUG")
	set(CMAKE_C_FLAGS_DEBUG "/MTd /Zi /Ob0 /Od /RTC1")
	set(CMAKE_C_FLAGS "/DWIN32 /D_WINDOWS /W3")
	message(STATUS "LIBRARY_OUTPUT_PATH ${LIBRARY_OUTPUT_PATH}")
	message(STATUS "CMAKE_CXX_FLAGS ${CMAKE_CXX_FLAGS}")
	message(STATUS "CMAKE_CXX_FLAGS_DEBUG ${CMAKE_CXX_FLAGS_DEBUG}")
	message(STATUS "CMAKE_CXX_FLAGS_RELEASE ${CMAKE_CXX_FLAGS_RELEASE}")
	message(STATUS "CMAKE_C_FLAGS ${CMAKE_C_FLAGS}")
	message(STATUS "CMAKE_C_FLAGS_DEBUG ${CMAKE_C_FLAGS_DEBUG}")
	message(STATUS "CMAKE_C_FLAGS_RELEASE ${CMAKE_C_FLAGS_RELEASE}")
	
	add_library(lib${PROJECT_NAME} ${LuaLibModule})
	add_executable(${PROJECT_NAME}Compiler ${LuaCompilerModule})
	
	set_target_properties(lib${PROJECT_NAME} PROPERTIES OUTPUT_NAME "lib${PROJECT_NAME}")
	set_target_properties(${PROJECT_NAME}Compiler PROPERTIES OUTPUT_NAME "xcompiler")
ENDIF()