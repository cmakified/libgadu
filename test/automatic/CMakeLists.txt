add_executable (connect
	connect.c)
target_link_libraries (connect gadu)

add_executable (endian1
	endian1.c)
target_link_libraries (endian1 gadu)

add_executable (message1
	message1.c)
target_link_libraries (message1 gadu)

add_executable (packet
	packet.c)
target_link_libraries (packet gadu)

add_executable (resolver
	resolver.c)
target_link_libraries (resolver gadu)


add_executable (convert
	convert.c)
target_link_libraries (convert gadu)

add_executable (hash
	hash.c)
target_link_libraries (hash gadu)

add_executable (message2
	message2.c)
target_link_libraries (message2 gadu)

find_package (Perl)
if (PERL_FOUND)
	include_directories (script)
	set (scripts_file ${CMAKE_CURRENT_BINARY_DIR}/scripts.c)
	add_custom_command (OUTPUT ${scripts_file}
		COMMAND ${CMAKE_COMMAND} -E remove -f ${scripts_file}
		COMMAND ${CMAKE_COMMAND} -E chdir
			${CMAKE_CURRENT_SOURCE_DIR}/script
			${PERL_EXECUTABLE} compile
			00-login.scr
			05-multilogon.scr
			10-contacts.scr
			12-user_data.scr
			20-messages.scr
			25-images.scr
			30-status.scr
			40-userlist.scr
			50-pubdir.scr
			90-xml.scr
			95-ping.scr
			99-logoff.scr > ${scripts_file})
	add_executable (protocol
		${scripts_file}
		protocol.c)
	target_link_libraries (protocol gadu)
endif ()