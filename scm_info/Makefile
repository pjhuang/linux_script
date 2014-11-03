# 
# Note:
#       If this is the first time to build images, invoke 'make' or 'make all' 
#       to build images for default target.
#       
#       To build custom targets other than default, invoke 'make TARGET_config' to set  
#       system to TARGET configurations, and then invoke 'make' to build images.
#       Valid TARGETs are the files TARGET.mk in configs/ directory.
#
# Try:
#     'make'                --> regular make to build images.
#     'make all'            --> reconstruct rootfs directory and build images.
#
#     'make clean'          --> clean all object files in source tree.
#     'make distclean'      --> remove all generated files and current configuration.
#
#     'make install'        --> install kernel modules, libraries and programs to root filesystem directory.
#     
#     'make TARGET_config   --> set to TARGET configurations, valid TARGETs are TARGET.mk files in configs/.
#     
#     'make kernel'         --> build kernel, create uImage.
#     'make kernel_install' --> simply install kernel modules to root filesystem directory.
#     'make kernel_clean'   --> clean kernel object files.
#     
#     'make apps'           --> build 'apps/' sub-directory.
#     'make apps_install'   --> install 'apps/' files to root filesystem directory.
#     'make apps_clean'     --> clean 'apps/' object files.
#     
#     'make fs'             --> build 'apps/' and install to rootfs directory, then create filesystem image.
#     'make fs_img'         --> simply create filesystem image from root filesystem directory.
#     'make fs_init'        --> assign rootfs base directory and create target directory.
#     'make fs_clean'       --> remove filesystem images.

PROJECT_ROOT_DIR := $(shell pwd)
export PROJECT_ROOT_DIR

include common.mk
-include config.mk


DEFAULT_CONFIG_FILE  = io59.mk
DEFAULT_TARGET      := $(patsubst %.mk,%,$(DEFAULT_CONFIG_FILE))


# 
# The default target is simply to 'make' to generate kernel and filesystem images.
# 
target: setup
	@echo -e "#\n# Start building $(ATOP_MODEL) ...\n#"
	@$(MAKE) build
	@echo -e "#\n# Build $(ATOP_MODEL) successfully.\n#"


#
# restore rootfs and build.
#
all: setup
	@echo -e "#\n# Start building $(ATOP_MODEL) ...\n#"
	@$(MAKE) fs_init
	@$(MAKE) build
	@echo -e "#\n# Build $(ATOP_MODEL) successfully.\n#"

### ============================================================================

check_target: 
	@if [ ! -e target.mk ]; then \
        echo "Error. target.mk does not exist."; \
        exit 1; \
     fi


check: check_target
	@if [ ! -e config.txt ]; then \
        echo "Error. Target configuration is not done."; \
        exit 1; \
     fi


#
# If this is the first time to build and target is not specified, 
# use the the default target.
#

SCM_VER_FILE=.scmversion
SCM_URL_FILE=.scmurl

SCM_VER=`cat $(SCM_VER_FILE)`
SCM_URL=`cat $(SCM_URL_FILE)`

setup: 
	@if [ ! -e config.txt ] || [ ! -e target.mk ]; then \
        $(MAKE) $(DEFAULT_TARGET)_config; \
    fi
	@if [ -x tools/setlocalversion ]; then \
        tools/setlocalversion --save-scmversion; \
    fi
	@if [ -e $(SCM_VER_FILE) ]; then \
        echo "/* This file was generated/updated by $PROJECT_ROOT/tools/setlocalversion" >  configs/scm_info.h; \
        echo " * script file automatically. Typically, the script will be invoked at the" >> configs/scm_info.h; \
        echo " * build time." >> configs/scm_info.h; \
        echo " */" >> configs/scm_info.h; \
        echo "#define ATOP_SCM_VERSION \"$(SCM_VER)\"" >> configs/scm_info.h; \
    fi
	@if [ -e $(SCM_URL_FILE) ]; \
        then echo "#define ATOP_SCM_URL \"$(SCM_URL)\"" >> configs/scm_info.h; \
    fi


%_do-config: 
	@if [ ! -e target.mk ]; then \
        echo "Error. target.mk does not exist."; \
        exit 1; \
     fi
	@$(MAKE) TARGET_CONFIG=$* kernel_defconfig
	@$(MAKE) TARGET_CONFIG=$* fs_init
	@$(MAKE) TARGET_CONFIG=$* apps_init
	@echo "TARGET_CONFIG = $*" > config.mk
	@echo "done" > config.txt


# 
# Explicitly specify the target to config.
# 
%_config: 
	@if [ ! -e $(CONFIGS_DIR)/$*.mk ]; then \
        echo "Error. $(CONFIGS_DIR)/$*.mk does not exist."; \
        exit 1; \
     fi
	@echo -e "#\n#   Configure to $* ...\n#"
	@rm -f config.txt
	@rm -f config.mk
	@cp -f $(CONFIGS_DIR)/$*.mk target.mk
	@$(MAKE) $*_do-config

### ============================================================================

#
# the real target to build all.
#
build: check
	@$(MAKE) kernel
	@$(MAKE) apps
	@$(MAKE) install
	@$(MAKE) fs_img


install: check
	@$(MAKE) rootfs_check
	@$(MAKE) kernel_install
	@$(MAKE) apps_install


clean: 
	@$(MAKE) kernel_clean
	@$(MAKE) fs_clean
	@$(MAKE) apps_clean


distclean: 
	@echo -e "#\n#   Remove all generated files and configurations.\n#" 
	rm -f config.txt
	@$(MAKE) kernel_distclean
	@$(MAKE) fs_distclean
	@$(MAKE) apps_distclean
	rm -f config.mk
	rm -f target.mk
	@echo "#   distclean is done."

### ============================================================================

# 
# the linux output directory name of Makefile in KERNEL_DIR.
# 
LINUX_OUTPUT_DIR = output

# 
# the linux output directory of the target.
# 
LINUX_BUILD_DIR  = $(LINUX_OUTPUT_DIR).$(TARGET_CONFIG)


kernel: check
	@$(MAKE) -C $(KERNEL_DIR)
	@cp -af $(KERNEL_DIR)/output/arch/$(ARCH)/boot/uImage $(DOWNLOAD_DIR)/


kernel_defconfig: check_target
	@if [ -z "$(TARGET_CONFIG)" ]; then \
        echo "Error. TARGET_CONFIG is not defined."; \
        exit 1; \
     fi
	@if [ -n "$(LINUX_SOURCE_DIR)" ]; then \
        cd $(KERNEL_DIR); \
        if [ ! -d `readlink -m -n $(LINUX_SOURCE_DIR)` ]; then \
            echo "Error. $(LINUX_SOURCE_DIR) does not exist."; \
            exit 1; \
        fi; \
        linux_dir=$(LINUX_SOURCE_DIR); \
     else \
        linux_dir=$(LINUX_DEFAULT_DIR); \
     fi; \
     cd $(KERNEL_DIR); \
     if [ ! -e $(LINUX_SRC_DIR) ] || [ -h $(LINUX_SRC_DIR) ] || \
        [ "`readlink -m -n $(LINUX_SRC_DIR)`" != "`readlink -m -n $$linux_dir`" ]; then \
        rm -f $(LINUX_SRC_DIR); \
        ln -sf $$linux_dir $(LINUX_SRC_DIR); \
     fi
	@cd $(KERNEL_DIR); \
     rm -rf $(LINUX_BUILD_DIR); \
     mkdir -p $(LINUX_BUILD_DIR); \
     rm -rf $(LINUX_OUTPUT_DIR); \
     ln -sf $(LINUX_BUILD_DIR) $(LINUX_OUTPUT_DIR);
	@if [ -n $(LINUX_DEFCONFIG) ]; then \
        $(MAKE) -C $(KERNEL_DIR) $(LINUX_DEFCONFIG); \
     else \
        $(MAKE) -C $(KERNEL_DIR) $(LINUX_DEFAULT_CONFIG); \
     fi


kernel_install: check
	@$(MAKE) -C $(KERNEL_DIR) install


kernel_clean: 
	@$(MAKE) -C $(KERNEL_DIR) clean


kernel_distclean: 
	@$(MAKE) -C $(KERNEL_DIR) distclean
	@cd $(KERNEL_DIR); \
     if [ -h $(LINUX_SRC_DIR) ]; then \
        echo "rm -f $(LINUX_SRC_DIR)"; \
        rm -f $(LINUX_SRC_DIR); \
     fi
	cd $(KERNEL_DIR) && rm -rf $(LINUX_OUTPUT_DIR)
	cd $(KERNEL_DIR) && rm -rf $(LINUX_BUILD_DIR)

### ============================================================================

#
# check filesystem configuration.
#
rootfs_check: check
	@if [ ! -e $(ROOTFS_BASE_DIR) ] || [ ! -e $(ROOTFS_DIR) ]; then \
        echo "Error. Filesystem directories are not configured."; \
        exit 1; \
     fi


fs: check rootfs_check
	@$(MAKE) apps
	@$(MAKE) apps_install
	@$(MAKE) kernel_install
	@$(MAKE) fs_img


#
# simply create rootfs image from target root filesystem directory.
#
fs_img: check rootfs_check
	@$(MAKE) -C $(FILESYSTEM_DIR) PROJECT_ROOT_DIR=$(PROJECT_ROOT_DIR) fs_img
	@cp -af $(FILESYSTEM_DIR)/initrd.uboot $(DOWNLOAD_DIR)/


#
# assign rootfs base directory and create target directory.
#
fs_init: check_target
	@if [ -z "$(TARGET_CONFIG)" ]; then \
        echo "Error. TARGET_CONFIG is not defined."; \
        exit 1; \
     fi
	@if [ -z "$(TARGET_BASE_DIR)" ]; then \
        echo "Error. TARGET_BASE_DIR is not defined."; \
        exit 1; \
     fi
	@cd $(FILESYSTEM_DIR); \
     if [ ! -d `readlink -m -n "$(TARGET_BASE_DIR)"` ]; then \
        echo "Error. $(TARGET_BASE_DIR) does not exist."; \
        exit 1; \
     fi
	@cd $(FILESYSTEM_DIR); \
     if [ ! -e $(ROOTFS_BASE_DIR) ] || [ -h $(ROOTFS_BASE_DIR) ] || \
        [ "`readlink -m -n $(ROOTFS_BASE_DIR)`" != "`readlink -m -n $(TARGET_BASE_DIR)`" ]; then \
        rm -rf $(ROOTFS_BASE_DIR); \
        ln -sf $(TARGET_BASE_DIR) $(ROOTFS_BASE_DIR); \
     fi
	@cd $(FILESYSTEM_DIR); \
     rm -rf $(ROOTFS_DIR) target.$(TARGET_CONFIG); \
     mkdir -p target.$(TARGET_CONFIG); \
     ln -sf target.$(TARGET_CONFIG) $(ROOTFS_DIR);
	@$(MAKE) -C $(FILESYSTEM_DIR) PROJECT_ROOT_DIR=$(PROJECT_ROOT_DIR) init


fs_clean: 
	@$(MAKE) -C $(FILESYSTEM_DIR) PROJECT_ROOT_DIR=$(PROJECT_ROOT_DIR) clean


fs_distclean: 
	@$(MAKE) -C $(FILESYSTEM_DIR) PROJECT_ROOT_DIR=$(PROJECT_ROOT_DIR) distclean
	@cd $(FILESYSTEM_DIR) && rm -rf $(ROOTFS_DIR) target.$(TARGET_CONFIG)
	@if [ -h $(ROOTFS_BASE_DIR) ]; then \
        rm -f $(ROOTFS_BASE_DIR); \
     fi

### ============================================================================

apps: check
	@$(MAKE) -C $(APPS_DIR) PROJECT_ROOT_DIR=$(PROJECT_ROOT_DIR)


apps_init: check_target
	@if [ -z "$(TARGET_CONFIG)" ]; then \
        echo "Error. TARGET_CONFIG is not defined."; \
        exit 1; \
     fi


apps_install: check
	@$(MAKE) -C $(APPS_DIR) PROJECT_ROOT_DIR=$(PROJECT_ROOT_DIR) install


apps_clean: 
	@$(MAKE) -C $(APPS_DIR) PROJECT_ROOT_DIR=$(PROJECT_ROOT_DIR) clean


apps_distclean: 
	@$(MAKE) -C $(APPS_DIR) PROJECT_ROOT_DIR=$(PROJECT_ROOT_DIR) distclean


### ============================================================================

.PHONY: all install clean build check check_target \
        kernel kernel_defconfig kernel_install kernel_clean kernel_distclean \
        apps apps_init apps_install apps_clean apps_distclean \
        fs fs_init rootfs_check fs_img fs_clean fs_distclean


