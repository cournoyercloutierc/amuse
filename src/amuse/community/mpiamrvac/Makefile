ifeq ($(origin AMUSE_DIR), undefined)
  AMUSE_DIR := $(shell amusifier --get-amuse-dir)
endif
-include $(AMUSE_DIR)/config.mk

# need := to expand the variable here and not when it is used
FORTRAN:=$(FC) 
MPIFC ?= mpif90
PYTHON ?= python
FC      = $(MPIFC)
LDFLAGS  += -lm $(MUSE_LD_FLAGS)

FFLAGS += $(FCFLAGS)

# check for gfortran, this catches link and compiler wrappers
ifeq ($(findstring GNU, $(shell $(FC) --version)), GNU)
FFLAGS += -std=legacy -ffree-form
endif


# xlf flags, untested, need to add flags for free form maybe
ifeq ($(findstring xlf, $(notdir $(FORTRAN))), xlf)
FFLAGS += -ffree-form
F77FLAGS += -qfixed
endif

# ifort flags
IFORT_MAJOR_MINOR=
ifeq ($(findstring ifort, $(notdir $(FORTRAN))), ifort)
IFORT_VERSION?=$(shell $(FC) -v 2>&1 | cut -d\  -f2)
IFORT_MAJOR_MINOR=$(shell echo $(IFORT_VERSION) | cut -s -d'.' -f1,2)
FFLAGS += -FR
endif


BUILDDIR = build
TWOD_BUILDDIR = build_2d
ONED_BUILDDIR = build_1d
TWOD_BUILDDIR_ACC = build_2d_acc
ONED_BUILDDIR_ACC = build_1d_acc
BUILDDIR_ACC = build_acc

OBJS = $(BUILDDIR)/interface.o
OBJSACC = $(BUILDDIR_ACC)/interface.o
OBJS2D = $(TWOD_BUILDDIR)/interface.o
OBJS2DACC = $(TWOD_BUILDDIR_ACC)/interface.o
OBJS1D = $(ONED_BUILDDIR)/interface.o
OBJS1DACC = $(ONED_BUILDDIR_ACC)/interface.o

LIBNAME = libmpiamrvac.a

CODEDIR = src/mpiamrvac/src

CODELIB = $(BUILDDIR)/$(LIBNAME)
CODELIB1D = $(ONED_BUILDDIR)/$(LIBNAME)
CODELIB2D = $(TWOD_BUILDDIR)/$(LIBNAME)
CODELIB2DACC = $(TWOD_BUILDDIR_ACC)/$(LIBNAME)
CODELIB1DACC = $(ONED_BUILDDIR_ACC)/$(LIBNAME)
CODELIBACC = $(BUILDDIR_ACC)/$(LIBNAME)


DOWNLOAD_FROM_WEB = $(PYTHON) ./download.py
PATCH_FILES = $(PYTHON) ./patch_files.py
PATCH_FILES_IFORT91 = $(PYTHON) ./patch_files_ifort91.py

all: mpiamrvac_worker 

ifdef DOWNLOAD_CODES
$(CODEDIR)/makefile:
	make -C . download
else
$(CODEDIR)/makefile:
	@echo ""
	@echo "!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!"
	@echo ""
	@echo "DOWNLOAD_CODES is not set. MpiAmrVac will not be downloaded and build."
	@echo "If you do want MpiAmrVac, set DOWNLOAD_CODES to 1."
	@echo "bash> export DOWNLOAD_CODES=1"
	@echo "csh> setenv DOWNLOAD_CODES 1"
	@echo ""
	@echo "!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!"
	@echo ""
	@make -s --no-print-directory -C . raise_error
endif

download:
	$(RM) -Rf .pc
	$(RM) -Rf src
	mkdir src
	$(DOWNLOAD_FROM_WEB)
	$(PATCH_FILES)
ifeq ($(IFORT_MAJOR_MINOR), 9.1)
	$(info $(FFLAGS))
	$(PATCH_FILES_IFORT91)
endif


clean:
	$(RM) -f *.so *.o *.pyc worker_code.cc worker_code.h *.mod
	$(RM) *~ mpiamrvac_worker worker_code.f90
	$(RM) -f mpiamrvac_worker_2d mpiamrvac_worker_1d
	$(RM) -f mpiamrvac_worker_2dacc
	$(RM) -Rf mpiamrvac_worker*.dSYM
	$(RM) -f amrvac
	-make -C $(CODEDIR) clean
	$(RM) -Rf $(BUILDDIR)
	$(RM) -Rf $(TWOD_BUILDDIR)
	$(RM) -Rf $(ONED_BUILDDIR)
	$(RM) -Rf $(TWOD_BUILDDIR_ACC)
	$(RM) -Rf $(ONED_BUILDDIR_ACC)
	$(RM) -Rf $(BUILDDIR_ACC)
	$(RM) -Rf *.dSYM
	
distclean:
	$(RM) -f *.so *.o *.pyc worker_code.cc worker_code.h *.mod
	$(RM) *~ mpiamrvac_worker worker_code.f90
	$(RM) -f mpiamrvac_worker_2d mpiamrvac_worker_1d
	$(RM) -f mpiamrvac_worker_2dacc
	$(RM) -f amrvac
	$(RM) -Rf src
	$(RM) -Rf src.*
	$(RM) -Rf .pc
	$(RM) -Rf $(BUILDDIR)
	$(RM) -Rf $(TWOD_BUILDDIR)
	$(RM) -Rf $(ONED_BUILDDIR)
	$(RM) -Rf $(TWOD_BUILDDIR_ACC)
	$(RM) -Rf $(ONED_BUILDDIR_ACC)
	$(RM) -Rf $(BUILDDIR_ACC)
	$(RM) -Rf mpiamrvac_worker_*.dSYM

$(BUILDDIR)/makefile: $(CODEDIR)/makefile
	cp -r $(CODEDIR) $(BUILDDIR)
	
$(BUILDDIR_ACC)/makefile: $(CODEDIR)/makefile
	cp -r $(CODEDIR) $(BUILDDIR_ACC)

$(TWOD_BUILDDIR)/makefile: $(CODEDIR)/makefile
	cp -r $(CODEDIR) $(TWOD_BUILDDIR)
    
$(TWOD_BUILDDIR_ACC)/makefile: $(CODEDIR)/makefile
	cp -r $(CODEDIR) $(TWOD_BUILDDIR_ACC)
	
$(ONED_BUILDDIR)/makefile: $(CODEDIR)/makefile
	cp -r $(CODEDIR) $(ONED_BUILDDIR)
	
$(ONED_BUILDDIR_ACC)/makefile: $(CODEDIR)/makefile
	cp -r $(CODEDIR) $(ONED_BUILDDIR_ACC)
    
$(CODELIB): $(BUILDDIR)/makefile
	cd $(BUILDDIR); ./setamrvac -d=33 -g=14,14,14 -phi=3 -z=2 -p=hd -u=amuse -cp=openmpi
	make -C $(BUILDDIR) clean amrvac $(LIBNAME) F90FLAGS="$(FFLAGS)" F90="$(MPIFC)"

$(CODELIBACC): $(BUILDDIR_ACC)/makefile
	cd $(BUILDDIR_ACC); ./setamrvac -d=33 -g=14,14,14 -phi=3 -z=2 -p=hdacc -u=amuse -cp=openmpi
	make -C $(BUILDDIR_ACC) clean amrvac $(LIBNAME) F90FLAGS="$(FFLAGS)" F90="$(MPIFC)"

$(CODELIB2D): $(TWOD_BUILDDIR)/makefile
	cd $(TWOD_BUILDDIR); ./setamrvac -d=22 -g=14,14 -phi=3 -z=2 -p=hd -u=amuse -cp=openmpi
	make -C $(TWOD_BUILDDIR) clean amrvac $(LIBNAME) F90FLAGS="$(FFLAGS)" F90="$(MPIFC)"
	
$(CODELIB2DACC): $(TWOD_BUILDDIR_ACC)/makefile
	cd $(TWOD_BUILDDIR_ACC); ./setamrvac -d=22 -g=14,14 -phi=3 -z=2 -p=hdacc -u=amuse -cp=openmpi
	make -C $(TWOD_BUILDDIR_ACC) clean amrvac $(LIBNAME) F90FLAGS="$(FFLAGS)" F90="$(MPIFC)"
	
$(CODELIB1D): $(ONED_BUILDDIR)/makefile
	cd $(ONED_BUILDDIR); ./setamrvac -d=11 -g=14,14 -phi=3 -z=2 -p=hd -u=amuse -cp=openmpi
	make -C $(ONED_BUILDDIR) clean amrvac $(LIBNAME) F90FLAGS="$(FFLAGS)" F90="$(MPIFC)"
	
$(CODELIB1DACC): $(ONED_BUILDDIR_ACC)/makefile
	cd $(ONED_BUILDDIR_ACC); ./setamrvac -d=11 -g=14,14 -phi=3 -z=2 -p=hdacc -u=amuse -cp=openmpi
	make -C $(ONED_BUILDDIR_ACC) clean amrvac $(LIBNAME) F90FLAGS="$(FFLAGS)" F90="$(MPIFC)"
	
worker_code.f90: interface.py
	$(CODE_GENERATOR) --type=f90 interface.py MpiAmrVacInterface -o $@

mpiamrvac_worker: worker_code.f90 $(CODELIB) $(OBJS)
	$(MPIFC) $(CXXFLAGS) $(SC_FLAGS) $(FS_FLAGS) $(LDFLAGS) -I$(BUILDDIR) $< $(OBJS) $(CODELIB) $(SC_MPI_FCLIBS) $(FS_LIBS) -o $@  $(LIBS)
	
mpiamrvac_worker_acc: worker_code.f90 $(CODELIBACC) $(OBJSACC)
	$(MPIFC) $(CXXFLAGS) $(SC_FLAGS) $(FS_FLAGS) $(LDFLAGS) -I$(BUILDDIR_ACC) $< $(OBJSACC) $(CODELIBACC) $(SC_MPI_FCLIBS) $(FS_LIBS) -o $@  $(LIBS)
	
mpiamrvac_worker_2d: worker_code.f90 $(CODELIB2D) $(OBJS2D)
	$(MPIFC) $(CXXFLAGS) $(SC_FLAGS) $(FS_FLAGS) $(LDFLAGS) -I$(TWOD_BUILDDIR) $< $(OBJS2D) $(CODELIB2D) $(SC_MPI_FCLIBS) $(FS_LIBS) -o $@  $(LIBS)
	
mpiamrvac_worker_2dacc: worker_code.f90 $(CODELIB2DACC) $(OBJS2DACC)
	$(MPIFC) $(CXXFLAGS) $(SC_FLAGS) $(FS_FLAGS) $(LDFLAGS) -I$(TWOD_BUILDDIR_ACC) $< $(OBJS2DACC) $(CODELIB2DACC) $(SC_MPI_FCLIBS) $(FS_LIBS) -o $@  $(LIBS)
	
mpiamrvac_worker_1d: worker_code.f90 $(CODELIB1D) $(OBJS1D)
	$(MPIFC) $(CXXFLAGS) $(SC_FLAGS) $(FS_FLAGS) $(LDFLAGS) -I$(ONED_BUILDDIR) $< $(OBJS1D) $(CODELIB1D) $(SC_MPI_FCLIBS) $(FS_LIBS) -o $@  $(LIBS)
	
mpiamrvac_worker_1dacc: worker_code.f90 $(CODELIB1DACC) $(OBJS1DACC)
	$(MPIFC) $(CXXFLAGS) $(SC_FLAGS) $(FS_FLAGS) $(LDFLAGS) -I$(ONED_BUILDDIR_ACC) $< $(OBJS1DACC) $(CODELIB1DACC) $(SC_MPI_FCLIBS) $(FS_LIBS) -o $@  $(LIBS)
	

$(BUILDDIR)/interface.f90: interface.t
	$(BUILDDIR)/vacpp.pl $< > $@
    
$(BUILDDIR)/interface.o: $(BUILDDIR)/interface.f90
	$(FC) $(FFLAGS) $(SC_FLAGS) -I$(BUILDDIR) -c -o $@ $<
    

$(TWOD_BUILDDIR)/interface.f90: interface.t
	$(TWOD_BUILDDIR)/vacpp.pl $< > $@
    
$(TWOD_BUILDDIR)/interface.o: $(TWOD_BUILDDIR)/interface.f90
	$(FC) $(FFLAGS) $(SC_FLAGS) -I$(TWOD_BUILDDIR) -c -o $@ $<
    
$(TWOD_BUILDDIR_ACC)/interface.f90: interface.t
	$(TWOD_BUILDDIR_ACC)/vacpp.pl $< > $@
    
$(TWOD_BUILDDIR_ACC)/interface.o: $(TWOD_BUILDDIR_ACC)/interface.f90
	$(FC) $(FFLAGS) $(SC_FLAGS) -I$(TWOD_BUILDDIR_ACC) -c -o $@ $<
    
$(ONED_BUILDDIR)/interface.f90: interface.t
	$(ONED_BUILDDIR)/vacpp.pl $< > $@
    
$(ONED_BUILDDIR)/interface.o: $(ONED_BUILDDIR)/interface.f90
	$(FC) $(FFLAGS) $(SC_FLAGS) -I$(ONED_BUILDDIR) -c -o $@ $<
	
$(ONED_BUILDDIR_ACC)/interface.f90: interface.t
	$(ONED_BUILDDIR_ACC)/vacpp.pl $< > $@
    
$(ONED_BUILDDIR_ACC)/interface.o: $(ONED_BUILDDIR_ACC)/interface.f90
	$(FC) $(FFLAGS) $(SC_FLAGS) -I$(ONED_BUILDDIR_ACC) -c -o $@ $<
    
$(BUILDDIR_ACC)/interface.f90: interface.t
	$(BUILDDIR_ACC)/vacpp.pl $< > $@
    
$(BUILDDIR_ACC)/interface.o: $(BUILDDIR_ACC)/interface.f90
	$(FC) $(FFLAGS) $(SC_FLAGS) -I$(BUILDDIR_ACC) -c -o $@ $<
	
%.o: %.f90
	$(FC) $(FFLAGS) -I$(CODEDIR) -c -o $@ $<
