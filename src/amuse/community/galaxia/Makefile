# standard amuse configuration include
# config.mk will be made after ./configure has run
ifeq ($(origin AMUSE_DIR), undefined)
  AMUSE_DIR := $(shell amusifier --get-amuse-dir)
endif
-include ${AMUSE_DIR}/config.mk

MPIFC ?= mpif90
FC ?= gfortran

FSLIB ?= -L$(AMUSE_DIR)/lib/forsockets -lforsockets 
FSINC ?= -I$(AMUSE_DIR)/lib/forsockets


OBJ	= interface.o

all:    GalaxyModel_worker

clean:
	rm -f  *~ *.pyc *.mod *.o GalaxyModel_worker worker_code.f90


$(OBJ): interface.f90
	$(FC) -c $(FCFLAGS) -o $@ $< 

GalaxyModel_worker:	worker_code.f90  $(OBJ)
	$(MPIFC) $(FCFLAGS) $(FS_FLAGS) $^ -o $@  $(FS_LIBS) $(LAPACK_LIBS) $(BLAS_LIBS)

.f90.o: $<
	$(FC) -c $(FCFLAGS) -o $@ $< 

worker_code.f90: interface.py
	$(CODE_GENERATOR) --type=f90 interface.py  BarAndSpiralsInterface -o $@

