#
# Copyright (C) 2011-2016 Intel Corporation. All rights reserved.
#
# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions
# are met:
#
#   * Redistributions of source code must retain the above copyright
#     notice, this list of conditions and the following disclaimer.
#   * Redistributions in binary form must reproduce the above copyright
#     notice, this list of conditions and the following disclaimer in
#     the documentation and/or other materials provided with the
#     distribution.
#   * Neither the name of Intel Corporation nor the names of its
#     contributors may be used to endorse or promote products derived
#     from this software without specific prior written permission.
#
# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS
# "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT
# LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR
# A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT
# OWNER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL,
# SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT
# LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE,
# DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY
# THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
# (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
# OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
#
#

######## SGX SDK Settings ########

SGX_SDK ?= /opt/intel/sgxsdk
SGX_MODE ?= HW 
SGX_ARCH ?= x64
SGX_DEBUG ?= 1
FLUSH_BUILD ?= 0

ifeq ($(SGX_MODE), HW)
   SGX_MODE_CFLAGS := -DSGX_MODE=1
endif

ifeq ($(FLUSH_BUILD), 1)
   SGX_FLUSH_CFLAGS := -DFLUSH_BUILD=1
endif

ifeq ($(shell getconf LONG_BIT), 32)
	SGX_ARCH := x86
else ifeq ($(findstring -m32, $(CXXFLAGS)), -m32)
	SGX_ARCH := x86
endif

ifeq ($(SGX_ARCH), x86)
	SGX_COMMON_CFLAGS := -m32
	SGX_LIBRARY_PATH := $(SGX_SDK)/lib
	SGX_ENCLAVE_SIGNER := $(SGX_SDK)/bin/x86/sgx_sign
	SGX_EDGER8R := $(SGX_SDK)/bin/x86/sgx_edger8r
else
	SGX_COMMON_CFLAGS := -m64
	SGX_LIBRARY_PATH := $(SGX_SDK)/lib64
	SGX_ENCLAVE_SIGNER := $(SGX_SDK)/bin/x64/sgx_sign
	SGX_EDGER8R := $(SGX_SDK)/bin/x64/sgx_edger8r
endif

ifeq ($(SGX_DEBUG), 1)
ifeq ($(SGX_PRERELEASE), 1)
$(error Cannot set SGX_DEBUG and SGX_PRERELEASE at the same time!!)
endif
endif

ifeq ($(SGX_DEBUG), 1)
        SGX_COMMON_CFLAGS += -O3 -g -D_FORTIFY_SOURCE=2 -fasynchronous-unwind-tables -fexceptions -fstack-protector-all -fstack-protector-strong -Wall -Werror=format-security -Werror=implicit-function-declaration 
else
        SGX_COMMON_CFLAGS += -O3 -D_FORTIFY_SOURCE=2 -fasynchronous-unwind-tables -fexceptions -fstack-protector-all -fstack-protector-strong -Wall -Werror=format-security -Werror=implicit-function-declaration 
endif

######## App Settings ########

ifneq ($(SGX_MODE), HW)
	Urts_Library_Name := sgx_urts_sim
else
	Urts_Library_Name := sgx_urts
endif

CDR_PATH := deps/sgx_zmq/
MBEDTLS_PATH := deps/mbedtls-SGX
MBEDTLS_LIB_PATH := $(MBEDTLS_PATH)/build/mbedtls_SGX-2.6.0/lib

App_Cpp_Files := App/dealer.cpp App/utils.cpp App/options.cpp
App_C_Files :=  App/ias-ra.c App/sgxsdk-ra-attester_u.c
Mbed_Tls_C_Files := deps/mbedtls-SGX/trusted/mbedtls-2.6.0/library/net_sockets.c

App_Include_Paths := -IApp -isystem $(SGX_SDK)/include -Iinclude -I$(MBEDTLS_PATH)/trusted/mbedtls-2.6.0/include -I$(CDR_PATH)/sgx_ucdr -I../../modules/rapidjson/include

App_C_Flags := $(SGX_COMMON_CFLAGS) -fPIC -Wno-attributes $(App_Include_Paths)

# Three configuration modes - Debug, prerelease, release
#   Debug - Macro DEBUG enabled.
#   Prerelease - Macro NDEBUG and EDEBUG enabled.
#   Release - Macro NDEBUG enabled.
ifeq ($(SGX_DEBUG), 1)
        App_C_Flags += -DDEBUG -UNDEBUG -UEDEBUG
else ifeq ($(SGX_PRERELEASE), 1)
        App_C_Flags += -DNDEBUG -DEDEBUG -UDEBUG
else
        App_C_Flags += -DNDEBUG -UEDEBUG -UDEBUG
endif

App_Cpp_Flags := $(App_C_Flags) -std=c++11
App_Link_Flags := $(SGX_COMMON_CFLAGS) -L$(SGX_LIBRARY_PATH) -L$(MBEDTLS_LIB_PATH) -lsgx_uprotected_fs -L$(CDR_PATH)/sgx_ucdr -lsgx_uprotected_cdr -l$(Urts_Library_Name) -lpthread  -L./App/ -lmbedtls_SGX_u -lzmq -lssl -lcrypto -lcurl

ifneq ($(SGX_MODE), HW)
	App_Link_Flags += -lsgx_uae_service_sim
else
	App_Link_Flags += -lsgx_uae_service
endif

App_Cpp_Objects := $(App_Cpp_Files:.cpp=.o)
App_C_Objects := $(App_C_Files:.c=.o)
Mbed_Tls_C_Objects := $(Mbed_Tls_C_Files:.c=.o)

App_Name := dealer 

######## Enclave Settings ########

ifneq ($(SGX_MODE), HW)
	Trts_Library_Name := sgx_trts_sim
	Service_Library_Name := sgx_tservice_sim
else
	Trts_Library_Name := sgx_trts 
	Service_Library_Name := sgx_tservice
endif
Crypto_Library_Name := sgx_tcrypto

Enclave_Cpp_Files := Enclave/ecalls.cpp Enclave/ssl_conn_hdlr.cpp Enclave/helper.cpp Enclave/crypto.cpp Enclave/memmem.cpp Enclave/ra-challenger.cpp Enclave/kms_client.cpp
Enclave_C_Files :=   Enclave/mbedtls-ra-attester.c Enclave/sgxsdk-ra-attester_t.c Enclave/ecalls.c 
Enclave_Include_Paths := -I$(MBEDTLS_PATH)/trusted/mbedtls-2.6.0/include  -Iinclude -IEnclave \
	-isystem $(SGX_SDK)/include -isystem $(SGX_SDK)/include/tlibc -isystem $(SGX_SDK)/include/stlport

Enclave_C_Flags := $(SGX_COMMON_CFLAGS) $(SGX_FLUSH_CFLAGS) $(SGX_MODE_CFLAGS) -nostdinc -fvisibility=hidden -fpie -fstack-protector $(Enclave_Include_Paths)
Enclave_Cpp_Flags := $(Enclave_C_Flags) -std=c++03 -nostdinc++ -fpermissive

# To generate a proper enclave, it is recommended to follow below guideline to link the trusted libraries:
#    1. Link sgx_trts with the `--whole-archive' and `--no-whole-archive' options,
#       so that the whole content of trts is included in the enclave.
#    2. For other libraries, you just need to pull the required symbols.
#       Use `--start-group' and `--end-group' to link these libraries.
# Do NOT move the libraries linked with `--start-group' and `--end-group' within `--whole-archive' and `--no-whole-archive' options.
# Otherwise, you may get some undesirable errors.
Enclave_Link_Flags := $(SGX_COMMON_CFLAGS) -Wl,--no-undefined -nostdlib -nodefaultlibs -nostartfiles -L$(MBEDTLS_LIB_PATH) -L$(SGX_LIBRARY_PATH) -L$(CDR_PATH)/sgx_tcdr/ \
	-Wl,--whole-archive -l$(Trts_Library_Name) -Wl,--no-whole-archive \
	-Wl,--start-group -lsgx_tstdc -lsgx_tstdcxx -lsgx_tcxx -l$(Crypto_Library_Name) -l$(Service_Library_Name) -lsgx_tcdr -lsgx_tprotected_fs  -lmbedtls_SGX_t -Wl,--end-group \
	-Wl,-Bstatic -Wl,-Bsymbolic -Wl,--no-undefined \
	-Wl,-pie,-eenclave_entry -Wl,--export-dynamic  \
	-Wl,--defsym,__ImageBase=0 \
	-Wl,--version-script=Enclave/Enclave.lds  -L./Enclave/. 

Enclave_Cpp_Objects := $(Enclave_Cpp_Files:.cpp=.o)
Enclave_C_Objects := $(Enclave_C_Files:.c=.o)

Enclave_Name := enclave.so
Signed_Enclave_Name := enclave.signed.so
Enclave_Config_File := Enclave/Enclave.config.xml

ifeq ($(SGX_MODE), HW)
ifneq ($(SGX_DEBUG), 1)
ifneq ($(SGX_PRERELEASE), 1)
Build_Mode = HW_RELEASE
endif
endif
endif


.PHONY: all run


ifeq ($(Build_Mode), HW_RELEASE)
all: $(App_Name) $(Enclave_Name)
	@echo "The project has been built in release hardware mode."
	@echo "Please sign the $(Enclave_Name) first with your signing key before you run the $(App_Name) to launch and access the enclave."
	@echo "To sign the enclave use the command:"
	@echo "   $(SGX_ENCLAVE_SIGNER) sign -key <your key> -enclave $(Enclave_Name) -out <$(Signed_Enclave_Name)> -config $(Enclave_Config_File)"
	@echo "You can also sign the enclave using an external signing tool."
	@echo "To build the project in simulation mode set SGX_MODE=SIM. To build the project in prerelease mode set SGX_PRERELEASE=1 and SGX_MODE=HW."
else
all: $(App_Name) $(Signed_Enclave_Name)
endif

run: all
ifneq ($(Build_Mode), HW_RELEASE)
	@$(CURDIR)/$(App_Name)
	@echo "RUN  =>  $(App_Name) [$(SGX_MODE)|$(SGX_ARCH), OK]"
endif

######## App Objects ########

#libmbedtls_sgx_u.a:
##	@echo "Please compile mbedtls-SGX and copy the libmbedtls_sgx_{t,u}.a here"
#	@exit 1

App/Enclave_u.c: $(SGX_EDGER8R) Enclave/Enclave.edl
	@cd App && $(SGX_EDGER8R) --untrusted ../Enclave/Enclave.edl --search-path ../Enclave --search-path $(SGX_SDK)/include
	@echo "GEN  =>  $@"

App/Enclave_u.o: App/Enclave_u.c
	@$(CC) $(App_C_Flags) -c $< -o $@
	@echo "CC   <=  $<"

App/%.o: App/%.c 
	@$(CC) $(App_C_Flags) -c $< -o $@
	@echo "CC  <=  $<"

deps/mbedtls-SGX/trusted/mbedtls-2.6.0/library/%.o: deps/mbedtls-SGX/trusted/mbedtls-2.6.0/library/net_sockets.c 
	@$(CC) $(App_C_Flags) -Ideps/mbedtls-SGX/trusted/mbedtls-2.6.0/include -c $< -o $@
	@echo "CC  <=  $<"

App/%.o: App/%.cpp
	@$(CXX) $(App_Cpp_Flags) -c $< -o $@
	@echo "CXX  <=  $<"

$(App_Name): App/Enclave_u.o deps/mbedtls-SGX/trusted/mbedtls-2.6.0/library/net_sockets.o $(App_Cpp_Objects) $(App_C_Objects) $(MBEDTLS_LIB_PATH)/libmbedtls_SGX_u.a
	@$(CXX) $^ -o $@ $(App_Link_Flags)
	@echo "LINK =>  $@"


######## Enclave Objects ########
# generate ca_bundle.h for enclave
ca_bundle:
	@echo "generating ca bundle for Enclave"
	cd Enclave && python embed_cert.py \@embed_cert.args

$(MBEDTLS_LIB_PATH)/libmbedtls_SGX_t.a:
	@echo "Please compile mbedtls-SGX before Dealer. libmbedtls_sgx_t.a is required from mbedtls-SGX"
	@exit 1

Enclave/Enclave_t.c: $(SGX_EDGER8R) Enclave/Enclave.edl
	@cd Enclave && $(SGX_EDGER8R) --trusted ../Enclave/Enclave.edl --search-path ../Enclave --search-path $(SGX_SDK)/include
	@echo "GEN  =>  $@"

Enclave/Enclave_t.o: Enclave/Enclave_t.c
	@$(CC) $(Enclave_C_Flags) -c $< -o $@
	@echo "CC   <=  $<"

Enclave/%.o: Enclave/%.cpp
	@$(CXX) $(Enclave_Cpp_Flags) -c $< -o $@
	@echo "CXX  <=  $<"

Enclave/%.o: Enclave/%.c
	@$(CC) $(Enclave_C_Flags) -c $< -o $@
	@echo "CC  <=  $<"


$(Enclave_Name): Enclave/Enclave_t.o $(Enclave_Cpp_Objects) $(Enclave_C_Objects) $(MBEDTLS_LIB_PATH)/libmbedtls_SGX_t.a
	@$(CXX) $^ -o $@ $(Enclave_Link_Flags)
	@echo "LINK =>  $@"

$(Signed_Enclave_Name): $(Enclave_Name)
	@$(SGX_ENCLAVE_SIGNER) sign -key Enclave/Enclave_private.pem -enclave $(Enclave_Name) -out $@ -config $(Enclave_Config_File)
	@echo "SIGN =>  $@"

.PHONY: clean

clean:
	@rm -f $(App_Name) $(Enclave_Name) $(Signed_Enclave_Name) $(App_Cpp_Objects)  \
         $(App_C_Objects) $(Mbed_Tls_C_Objects) App/Enclave_u.* $(Enclave_C_Objects) $(Enclave_Cpp_Objects) Enclave/Enclave_t.*
