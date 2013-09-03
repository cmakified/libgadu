add_executable (conn-async conn-async.c)
target_link_libraries (conn-async gadu)

add_executable (httphash httphash.c)
target_link_libraries (httphash gadu)

add_executable (register register.c)
target_link_libraries (register gadu)

add_executable (remind remind.c)
target_link_libraries (remind gadu)

add_executable (send send.c)
target_link_libraries (send gadu)

add_executable (status status.c)
target_link_libraries (status gadu)

add_executable (token token.c)
target_link_libraries (token gadu)

