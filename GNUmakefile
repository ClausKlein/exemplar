# Standard stuff

.SUFFIXES:

MAKEFLAGS+= --no-builtin-rules  # Disable the built-in implicit rules.
# MAKEFLAGS+= --warn-undefined-variables  # Warn when an undefined variable is referenced.

export hostSystemName=$(shell uname)

ifeq (${hostSystemName},Darwin)
  export LLVM_PREFIX:=$(shell brew --prefix llvm)
  export LLVM_DIR:=$(shell realpath ${LLVM_PREFIX})
  export PATH:=${LLVM_DIR}/bin:${PATH}
  CMAKE=cmake
  # XXX CMAKE?=/usr/local/bin/cmake

  # export CMAKE_CXX_STDLIB_MODULES_JSON:=${LLVM_DIR}/lib/c++/libc++.modules.json
  # export CXXFLAGS:=-stdlib=libc++
  # export LDFLAGS:=-L$(LLVM_DIR)/lib/c++ -lc++abi # XXX -lc++
  # export CXX:=clang++
  # export GCOV:="llvm-cov gcov"

  ### TODO: to test g++-16:
  export GCC_PREFIX:=$(shell brew --prefix gcc)
  export GCC_DIR:=$(shell realpath ${GCC_PREFIX})

  export CMAKE_CXX_STDLIB_MODULES_JSON:=${GCC_DIR}/lib/gcc/current/libstdc++.modules.json
  export CXXFLAGS:=-stdlib=libstdc++
  export CXX:=g++-16
  export GCOV:="gcov"
else ifeq (${hostSystemName},Linux)
  export LLVM_DIR:=/usr/lib/llvm-22
  export PATH:=${LLVM_DIR}/bin:${PATH}
  export CXX:=clang++-22
  CMAKE=cmake
endif

.PHONY: all install coverage gclean distclean format demo examples

all: build/compile_commands.json
	ln -sf $< .
	ninja -C build

build/compile_commands.json: CMakeLists.txt GNUmakefile
	${CMAKE} --version
	${CMAKE} -S . -B build -G Ninja \
	 -D BEMAN_USE_MODULES=YES \
	 -D BEMAN_USE_STD_MODULE=YES \
	 -D BEMAN_CYCLE_USE_MODULES=YES \
	 -D BEMAN_EXEMPLAR_USE_MODULES=YES \
	 -D CMAKE_CXX_STDLIB_MODULES_JSON=${CMAKE_CXX_STDLIB_MODULES_JSON} \
	 -D CMAKE_BUILD_TYPE=Release \
	 -D CMAKE_CXX_STANDARD=26 -D CMAKE_CXX_EXTENSIONS=NO -D CMAKE_CXX_STANDARD_REQUIRED=YES \
	 -D CMAKE_INSTALL_MESSAGE=LAZY \
	 -D CMAKE_PROJECT_TOP_LEVEL_INCLUDES=./infra/cmake/use-fetch-content.cmake \
	 --log-level=VERBOSE --fresh \
	 # -D CMAKE_CXX_FLAGS='-fno-inline --coverage' \
	 # --trace-expand --trace-source=use-fetch-content.cmake \
	 # --debug-find-pkg=GTest

install: build/cmake_install.cmake
	${CMAKE} --install build

examples: examples/CMakeLists.txt
	${CMAKE} -S examples -B examples/build -G Ninja \
	 -D BEMAN_USE_MODULES=YES \
	 -D BEMAN_USE_STD_MODULE=YES \
	 -D BEMAN_CYCLE_USE_MODULES=YES \
	 -D BEMAN_EXEMPLAR_USE_MODULES=YES \
	 -D CMAKE_CXX_STDLIB_MODULES_JSON=${CMAKE_CXX_STDLIB_MODULES_JSON} \
	 -D CMAKE_BUILD_TYPE=Release \
	 -D CMAKE_CXX_STANDARD=26 -D CMAKE_CXX_EXTENSIONS=NO -D CMAKE_CXX_STANDARD_REQUIRED=YES \
	 --log-level=VERBOSE --fresh
	ninja -C examples/build -v
	ninja -C examples/build test

# ==========================================================
CMakeUserPresets.json: cmake/CMakeUserPresets.json
	ln -s $< $@

release: build/$(hostSystemName)/release/compile_commands.json
	${CMAKE} --workflow --preset release
	touch $@

build/$(hostSystemName)/release/compile_commands.json: CMakeUserPresets.json CMakeLists.txt GNUmakefile
	${CMAKE} --version
	${CMAKE} --preset release --log-level=VERBOSE --fresh
	ln -fs build/$(hostSystemName)/release/compile_commands.json .
# ==========================================================

distclean: # XXX clean
	rm -rf build examples/build stagedir compile_commands.json
	find . -name '*~' -delete

gclean: clean
	find build -name '*.gc..' -delete

build/coverage: test
	mkdir -p $@

coverage: build/coverage
	gcovr --merge-mode-functions separate

format: distclean
	-pre-commit install
	-pre-commit autoupdate
	pre-commit run --all

demo: distclean
	${CMAKE} --preset appleclang-release --fresh --log-level=VERBOSE
	${CMAKE} --workflow appleclang-release
	${CMAKE} --preset gcc-release --fresh --log-level=VERBOSE
	${CMAKE} --workflow gcc-release
	${CMAKE} --preset llvm-release --fresh --log-level=VERBOSE
	${CMAKE} --workflow llvm-release
	# ${CMAKE} --preset llvm-debug --fresh --log-level=VERBOSE
	# ${CMAKE} --workflow llvm-debug


# Anything we don't know how to build will use this rule.
% ::
	ninja -C build $(@)
