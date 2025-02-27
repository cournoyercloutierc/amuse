ifeq ($(origin AMUSE_DIR), undefined)
  AMUSE_DIR := $(shell amusifier --get-amuse-dir)
endif
-include ${AMUSE_DIR}/config.mk

PYTHON ?= python

PYOPENGL_AVAILABLE := $(shell $(PYTHON) -c "import OpenGL"  1>&2 2> /dev/null && echo "yes" || echo "no")
PYIMAGE_AVAILABLE := $(shell $(PYTHON) -c "from PIL import Image"  1>&2 2> /dev/null && echo "yes" || echo "no")
PYOPENCL_AVAILABLE := $(shell $(PYTHON) -c "import pyopencl"  1>&2 2> /dev/null && echo "yes" || echo "no")
CFFI_AVAILABLE := $(shell $(PYTHON) -c "import cffi"  1>&2 2> /dev/null && echo "yes" || echo "no")




all: code build install test tupan_worker

code: src/setup.py

DOWNLOAD_CODES?=1

ifdef DOWNLOAD_CODES
src/setup.py:
	make -C . download
else
src/setup.py:
	@echo ""
	@echo "!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!"
	@echo ""
	@echo "DOWNLOAD_CODES is not set. tupan will not be downloaded and build."
	@echo "If you do want Athena, set DOWNLOAD_CODES to 1."
	@echo "bash> export DOWNLOAD_CODES=1"
	@echo "csh> setenv DOWNLOAD_CODES 1"
	@echo ""
	@echo "!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!"
	@echo ""
	@make -s --no-print-directory -C . raise_error
endif 

download:
	git clone -b master https://github.com/ggf84/tupan.git src

build:
	cp tupan.cfg ./src/tupan/
	(cd src; python setup.py build)

install:
	mkdir install
	(cd src; python setup.py install --prefix=../install)

test:
	@echo
	@echo "Testing import of modules required for tupan (pyopencl, cffi):"
ifeq ($(PYOPENCL_AVAILABLE),no)
	$(warning "Python import (optional) not available: pyopencl")
endif
ifeq ($(CFFI_AVAILABLE),no)
	$(error "Python imports not available: cffi")
endif
	@echo "Tests successful!"
	@echo


tupan_worker: interface.py
	$(CODE_GENERATOR) --type=py --mode=mpi -x interface TupanInterface TupanImplementation -o $@
	

clean:
	-cd src && python setup.py clean -a
	-rm -f *.pyc; 
	-rm -f tupan_worker tupan_worker_sockets
	-rm -rf install

distclean: clean
	-rm -f *.pyc
	-rm -rf src __pycache__

