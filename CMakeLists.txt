cmake_minimum_required(VERSION 3.10)
project(AddressBook CXX)

set(CMAKE_CXX_STANDARD 17)

# Usa la versione moderna (CONFIG). 
# Rispetto a 'find_package(Protobuf CONFIG REQUIRED)' non definisce la macro generate_proto_cpp, 
# che va sostituita con un comando personalizzato. Ma permette di linkare 
# protobuf::libprotobuf al posto di usare la variabile  ${Protobuf_LIBRARIES}, che ha l'enorme
# vantantaggio di risolvere automaticamente le dipendenze di protoc,
# come ad esempio le librerie di sistema necessarie per il linking. (non devo mettere a mano tutte 
# le librerie statiche di abseil ad esempio).
find_package(Protobuf CONFIG REQUIRED)

# File .proto
set(PROTO_FILE ${CMAKE_CURRENT_SOURCE_DIR}/addressbook.proto)

# File generati
set(GENERATED_PROTO_SRC ${CMAKE_CURRENT_BINARY_DIR}/addressbook.pb.cc)
set(GENERATED_PROTO_HDR ${CMAKE_CURRENT_BINARY_DIR}/addressbook.pb.h)

# Comando per generare i file
add_custom_command(
  OUTPUT ${GENERATED_PROTO_SRC} ${GENERATED_PROTO_HDR}
  COMMAND protobuf::protoc
    --proto_path=${CMAKE_CURRENT_SOURCE_DIR}
    --cpp_out=${CMAKE_CURRENT_BINARY_DIR}
    ${PROTO_FILE}
  DEPENDS ${PROTO_FILE}
  COMMENT "Generating C++ protocol buffer code from ${PROTO_FILE}"
  VERBATIM
)

# Definisci un custom target per esplicitare le dipendenze dai file generati
add_custom_target(generate_proto_cpp DEPENDS ${GENERATED_PROTO_SRC} ${GENERATED_PROTO_HDR})

# Eseguibile principale
add_executable(write.exe write.cpp ${GENERATED_PROTO_SRC} ${GENERATED_PROTO_HDR})
add_executable(read.exe read.cpp ${GENERATED_PROTO_SRC} ${GENERATED_PROTO_HDR})
add_dependencies(write.exe generate_proto_cpp)
add_dependencies(read.exe generate_proto_cpp)

target_include_directories(write.exe PRIVATE  ${CMAKE_CURRENT_BINARY_DIR})
target_link_libraries(write.exe PRIVATE protobuf::libprotobuf)
target_include_directories(read.exe PRIVATE  ${CMAKE_CURRENT_BINARY_DIR})
target_link_libraries(read.exe PRIVATE protobuf::libprotobuf)
