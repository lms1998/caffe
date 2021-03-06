# A makefile to build the docker images for caffe.
# Two caffe images will be built:
#   caffe:cpu --> A CPU-only build of caffe.
#   caffe:gpu --> A GPU-enabled build using the latest CUDA and CUDNN versions.

DOCKER ?= docker

ifneq ($(http_proxy),)
DOCKER_BUILD_ARG += --build-arg http_proxy=$(http_proxy)
endif
ifneq ($(https_proxy),)
DOCKER_BUILD_ARG += --build-arg https_proxy=$(https_proxy)
endif


all: docker_files standalone

.PHONY: standalone devel

standalone: cpu_standalone gpu_standalone intel_standalone

cpu_standalone: standalone/cpu/Dockerfile
	$(DOCKER) build $(DOCKER_BUILD_ARG) -t caffe:cpu standalone/cpu

gpu_standalone: standalone/gpu/Dockerfile
	$(DOCKER) build $(DOCKER_BUILD_ARG) -t caffe:gpu standalone/gpu

intel_standalone: standalone/intel/Dockerfile
	$(DOCKER) build $(DOCKER_BUILD_ARG) -t caffe:intel standalone/intel

docker_files: standalone_files

standalone_files: standalone/cpu/Dockerfile standalone/gpu/Dockerfile standalone/intel/Dockerfile

FROM_GPU = "nvidia/cuda:7.5-cudnn5-devel-ubuntu14.04"
FROM_CPU = "ubuntu:14.04"
GPU_CMAKE_ARGS = -DUSE_CUDNN=1
CPU_CMAKE_ARGS = -DCPU_ONLY=1

# A make macro to select the CPU or GPU base image.
define from_image
$(if $(strip $(findstring gpu,$@)),$(FROM_GPU),$(FROM_CPU))
endef

# A make macro to select the CPU or GPU build args.
define build_args
$(if $(strip $(findstring gpu,$@)),$(GPU_CMAKE_ARGS),$(CPU_CMAKE_ARGS))
endef

# A make macro to construct the CPU or GPU Dockerfile from the template
define create_docker_file
	@echo creating $@
	@echo "FROM "$(from_image) > $@
	@cat $^ | sed 's/$${CMAKE_ARGS}/$(build_args)/' >> $@
endef


standalone/%/Dockerfile: templates/Dockerfile.template
	$(create_docker_file)

