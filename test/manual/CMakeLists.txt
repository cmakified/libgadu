add_library (userconfig userconfig.c)

add_executable (client client.c)
target_link_libraries (client
	gadu
	userconfig)

add_executable (dcc7 dcc7.c)
target_link_libraries (dcc7
	gadu
	userconfig)

find_package (EXPAT)
find_package (CURL)
if (CURL_FOUND AND EXPAT_FOUND)
	include_directories (
		${EXPAT_INCLUDE_DIRS}
		${CURL_INCLUDE_DIRS}
		lib/)
	add_executable (search
		search.c
		lib/base64.c
		lib/http.c
		lib/oauth_parameter.c
		lib/urlencode
		lib/hmac.c
		lib/oauth.c
		lib/sha1.c
		lib/xml.c
	)
	target_link_libraries (search
		gadu
		userconfig
		${EXPAT_LIBRARIES}
		${CURL_LIBRARIES})
endif ()


add_executable (userlist userlist.c)
target_link_libraries (userlist
	gadu
	userconfig)

add_executable (voice7 voice7.c)
target_link_libraries (voice7
	gadu
	userconfig)
