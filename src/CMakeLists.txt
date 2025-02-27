# Create an object library so we only have to build pdb.c once when producing
# both static and shared library configurations.
add_library(pdb_object OBJECT pdb.c)
set_target_properties(pdb_object
  PROPERTIES
    POSITION_INDEPENDENT_CODE ON
    C_VISIBILITY_PRESET hidden)

if(PDB_ENABLE_ASSERTIONS)
    target_compile_definitions(pdb_object PRIVATE PDB_ENABLE_ASSERTIONS=1)
endif()

# Microsoft's cvinfo.h header requires c11 for anonymous struct/union
target_compile_features(pdb_object PUBLIC c_std_11)
target_include_directories(pdb_object PUBLIC "${PROJECT_SOURCE_DIR}/include")

# Note that cmake doesn't support specifying an object library in
# target_link_libraries until CMake 3.12, so we have to define the libpdb public
# interface in a separate interface library until we can upgrade from 3.10.2.
add_library(pdb_object_interface INTERFACE)
target_compile_features(pdb_object_interface
  INTERFACE
    $<TARGET_PROPERTY:pdb_object,INTERFACE_COMPILE_FEATURES>)
target_include_directories(pdb_object_interface
  INTERFACE
    $<TARGET_PROPERTY:pdb_object,INTERFACE_INCLUDE_DIRECTORIES>)

# Build libpdb.so and libpdb.a

add_library(pdb_static STATIC $<TARGET_OBJECTS:pdb_object>)
set_target_properties(pdb_static PROPERTIES OUTPUT_NAME pdb)
target_link_libraries(pdb_static PUBLIC pdb_object_interface)


