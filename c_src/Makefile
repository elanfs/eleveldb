LEVELDB_VSN ?= "2.0.38" # custom from https://github.com/basho/leveldb
SNAPPY_VSN ?= "1.1.9"
C_SRC_DIR := $(shell pwd)
PROJECT_DIR := $(abspath $(C_SRC_DIR)/..)
TARGET_DIR := $(PROJECT_DIR)/priv/

# LevelDB tools
TOOLS = \
	leveldb_repair \
	perf_dump \
	sst_rewrite \
	sst_scan

LDFLAGS := $(LDFLAGS) -lpthread
CFLAGS := $(CFLAGS) -I. -I $(C_SRC_DIRC_SRC_DIR)/leveldb/include -fPIC
CXXFLAGS := $(CXXFLAGS) -I. -I $(C_SRC_DIR)/leveldb/include -fPIC
TEST_CXXFLAGS := $(CXXFLAGS) -Wno-narrowing

ifeq ($(shell uname -s), Darwin)

	# OSX with homebrew
  HAS_BREW := $(shell command -v brew;)

	ifdef HAS_BREW
		SNAPPY_DIR ?= $(shell brew --prefix snappy)
	endif

	# Default dir (Mac ports)
	SNAPPY_DIR ?= /usr/local/opt/snappy

  LDFLAGS += -L${SNAPPY_DIR}/lib

	# Resolves C++11 narrowing error on Mac OS
	TEST_CXXFLAGS += -Wno-c++11-narrowing
endif

get-deps:
	git config --global --add safe.directory /__w/eleveldb/eleveldb
	echo "ubuntu-latest image with otp-22, are you happy now?"
	if [ ! -r snappy-$(SNAPPY_VSN).tar.gz ]; then \
		wget -O snappy-$(SNAPPY_VSN).tar.gz https://github.com/google/snappy/archive/refs/tags/$(SNAPPY_VSN).tar.gz; \
		tar -xzf snappy-$(SNAPPY_VSN).tar.gz
	fi	
	if [ ! -d leveldb ]; then \
	    git clone --depth=1 --branch=$(LEVELDB_VSN) https://github.com/basho/leveldb && \
	    (cd leveldb && git submodule update --depth 1 --init); \
	fi

compile: snappy ldb
	for tool in $(TOOLS); do cp leveldb/$$tool $(TARGET_DIR); done

ldb:
	@echo "Building LevelDB..."
	$(MAKE) LDFLAGS="" -C leveldb all
	$(MAKE) LDFLAGS="$(LDFLAGS)" -C leveldb tools

snappy: system/lib/libsnappy.a

system/lib/libsnappy.a:
	(cd snappy-$(SNAPPY_VSN) && \
	 if [ -r autogen.sh ]; then \
	   ./autogen.sh && ./configure --prefix=$(C_SRC_DIR)/system && $(MAKE) && $(MAKE) install; \
	 else \
	   mkdir -p build && cd build && \
	   mkdir -p $(C_SRC_DIR)/system && \
	   cmake -D SNAPPY_BUILD_TESTS=0 -D SNAPPY_BUILD_BENCHMARKS=0 \
	         -D CMAKE_INSTALL_PREFIX=$(C_SRC_DIR)/system \
	     ..; \
	  fi && \
	  $(MAKE) && $(MAKE) install)
	mv system/lib64 system/lib || true

clean:
	$(MAKE) -C leveldb clean
	for tool in $(TOOLS); do rm -f $(TARGET_DIR)$$tool; done
	rm -rf system snappy-$(SNAPPY_VSN)/build

test: compile
	$(MAKE) LDFLAGS="$(LDFLAGS)" CXXFLAGS="$(TEST_CXXFLAGS)" CFLAGS="$(CFLAGS)" -C leveldb test
