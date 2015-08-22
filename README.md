# cpm-flatbuffers
[CPM](http://cpm.rocks) external for
[Flatbuffers](https://github.com/google/flatbuffers/).

## Usage

Add this line to your `CMakeLists.txt` file:
```
CPM_AddModule("cpm-flatbuffers"
  GIT_REPOSITORY "https://github.com/cogumbreiro/cpm-flatbuffers")
```

This module exports CMake function `compile_flatbuffers_schema_to_cpp` that generates C++ headers from a Flatbuffers schema. The first parameter is the schema file; the second parameter is the CMake target (used so we can add dependencies). For instance,

    compile_flatbuffers_schema_to_cpp(src/monster.fbs monster)

generates file `src/monster_generated.h` set to target `monster`.
