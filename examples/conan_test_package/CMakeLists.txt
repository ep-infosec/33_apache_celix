#  Licensed to the Apache Software Foundation (ASF) under one
#  or more contributor license agreements.  See the NOTICE file
#  distributed with this work for additional information
#  regarding copyright ownership.  The ASF licenses this file
#  to you under the Apache License, Version 2.0 (the
#  "License"); you may not use this file except in compliance #  with the License.  You may obtain a copy of the License at
#
#    http://www.apache.org/licenses/LICENSE-2.0
#
#   Unless required by applicable law or agreed to in writing,
#   software distributed under the License is distributed on an
#   "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
#   KIND, either express or implied.  See the License for the
#   specific language governing permissions and limitations
#   under the License.

cmake_minimum_required (VERSION 3.14)
project(test_package)

set(CMAKE_CXX_STANDARD 17)

find_package(Celix REQUIRED)

add_celix_bundle(hello VERSION 1.0.0 SOURCES hello_bundle.c)
target_link_libraries(hello PRIVATE Celix::log_helper)
celix_get_bundle_file(hello HELLO_TEST_BUNDLE)

add_executable(use_framework test_framework.c)
target_link_libraries(use_framework Celix::framework)
add_celix_bundle_dependencies(use_framework hello)
target_compile_definitions(use_framework PRIVATE HELLO_TEST_BUNDLE_LOCATION="${HELLO_TEST_BUNDLE}")

option(TEST_HTTP_ADMIN "Test http_admin" OFF)
if (TEST_HTTP_ADMIN)
    add_celix_bundle(http_admin_sut
            SOURCES
            test_http_admin_activator.c
            VERSION 1.0.0
            )
    target_link_libraries(http_admin_sut PRIVATE Celix::http_admin_api)
    add_celix_container(use_http_admin COPY
            BUNDLES
            Celix::http_admin
            http_admin_sut
            )
endif ()

option(TEST_LOG_SERVICE "Test log_admin" OFF)
if (TEST_LOG_SERVICE)
    add_celix_bundle(my_log_writer
            SOURCES
            my_log_writer_activator.c
            VERSION 1.0.0
            )
    target_link_libraries(my_log_writer PRIVATE Celix::log_helper Celix::log_service_api)
    add_celix_container(use_log_writer COPY)
    celix_container_bundles(use_log_writer LEVEL 0 Celix::log_admin)
    celix_container_bundles(use_log_writer LEVEL 1 my_log_writer)
    celix_container_bundles(use_log_writer LEVEL 2 hello)
endif ()

option(TEST_SYSLOG_WRITER "Test syslog_writer" OFF)
if (TEST_SYSLOG_WRITER)
    add_celix_container(use_syslog_writer COPY)
    celix_container_bundles(use_syslog_writer LEVEL 0 Celix::log_admin)
    celix_container_bundles(use_syslog_writer LEVEL 1 Celix::syslog_writer)
    celix_container_bundles(use_syslog_writer LEVEL 2 hello)
endif ()

option(TEST_PUBSUB "Test PubSub" OFF)
if (TEST_PUBSUB)
    add_celix_bundle(my_pubsub_admin
            SOURCES
            my_psa_activator.c
            )
    target_link_libraries(my_pubsub_admin PRIVATE Celix::framework Celix::dfi Celix::log_helper Celix::utils)
    target_link_libraries(my_pubsub_admin PRIVATE Celix::pubsub_spi Celix::pubsub_utils )
    add_celix_container(use_my_psa
            BUNDLES
            Celix::celix_pubsub_serializer_json
            Celix::celix_pubsub_serializer_avrobin
            Celix::celix_pubsub_topology_manager
            my_pubsub_admin
            hello
            PROPERTIES
            PUBSUB_TOPOLOGY_MANAGER_VERBOSE=true
            )
endif ()

option(TEST_PSA_ZMQ "Test ZeroMQ PubSub Admin" OFF)
if (TEST_PSA_ZMQ)
    add_celix_container(use_psa_zmq
            BUNDLES
            Celix::celix_pubsub_serializer_json
            Celix::celix_pubsub_serializer_avrobin
            Celix::celix_pubsub_topology_manager
            Celix::celix_pubsub_admin_zmq
            hello
            PROPERTIES
            PUBSUB_TOPOLOGY_MANAGER_VERBOSE=true
            )
endif ()

option(TEST_PSA_TCP "Test TCP PubSub Admin" OFF)
if (TEST_PSA_TCP)
    add_celix_container(use_psa_tcp
            BUNDLES
            Celix::celix_pubsub_serializer_json
            Celix::celix_pubsub_topology_manager
            Celix::celix_pubsub_admin_tcp
            hello
            PROPERTIES
            PSA_TCP_VERBOSE=true
            PUBSUB_TOPOLOGY_MANAGER_VERBOSE=true
            )
endif ()

option(TEST_PSA_UDP_MC "Test UDP MC PubSub Admin" OFF)
if (TEST_PSA_UDP_MC)
    add_celix_container(use_psa_udp_mc
            BUNDLES
            Celix::celix_pubsub_serializer_json
            Celix::celix_pubsub_topology_manager
            Celix::celix_pubsub_admin_udp_multicast
            hello
            PROPERTIES
            PSA_UDPMC_VERBOSE=true
            PUBSUB_TOPOLOGY_MANAGER_VERBOSE=true
            )
endif ()

option(TEST_PSA_WS "Test WebSocket PubSub Admin" OFF)
if (TEST_PSA_WS)
    add_celix_container(use_psa_ws
            BUNDLES
            Celix::http_admin
            Celix::celix_pubsub_serializer_json
            Celix::celix_pubsub_topology_manager
            Celix::celix_pubsub_admin_websocket
            hello
            PROPERTIES
            PUBSUB_TOPOLOGY_MANAGER_VERBOSE=true
            )
endif ()

option(TEST_PSA_DISCOVERY_ETCD "Test the PubSub Discovery (ETCD) bundle" OFF)
if (TEST_PSA_DISCOVERY_ETCD)
    add_celix_container(use_psa_discovery_etcd
            BUNDLES
            Celix::celix_pubsub_serializer_json
            Celix::celix_pubsub_topology_manager
            Celix::celix_pubsub_discovery_etcd
            my_pubsub_admin
            hello
            PROPERTIES
            PUBSUB_TOPOLOGY_MANAGER_VERBOSE=true
            )
endif ()

option(TEST_RSA "Test Remote Service Admin" OFF)
if (TEST_RSA)
    add_celix_bundle(my_rsa
            SOURCES
            my_rsa_activator.c
            )
    target_link_libraries(my_rsa PRIVATE
            Celix::log_helper
            Celix::deprecated_rsa_spi
            Celix::rsa_common
            Celix::framework
            )
    add_celix_container(use_my_rsa
            BUNDLES
            Celix::rsa_topology_manager
            my_rsa
            hello
            PROPERTIES
            )
            
    add_executable(use_c_rsa_spi test_c_rsa_spi.c)
    target_link_libraries(use_c_rsa_spi PRIVATE Celix::c_rsa_spi)
endif ()

option(TEST_RSA_DFI "Test the Remote Service Admin Service DFI" OFF)
if (TEST_RSA_DFI)
    add_celix_container(use_rsa_dfi
            BUNDLES
            Celix::rsa_topology_manager
            Celix::rsa_dfi
            hello
            PROPERTIES
            )
endif ()

option(TEST_RSA_SHM_V2 "Test the Remote Service Admin SHM V2" OFF)
if (TEST_RSA_SHM_V2 AND CMAKE_SYSTEM_NAME STREQUAL "Linux")
    add_celix_container(use_rsa_shm_v2
            BUNDLES
            Celix::rsa_topology_manager
            Celix::rsa_shm
            hello
            PROPERTIES
            )
endif ()

option(TEST_RSA_RPC_JSON "Test the Remote Service Admin Json RPC" OFF)
if (TEST_RSA_RPC_JSON)
    add_celix_container(use_rsa_rpc_json
            BUNDLES
            Celix::rsa_json_rpc
            hello
            PROPERTIES
            )
endif ()

option(TEST_RSA_DISCOVERY_CONFIGURED "Test the Discovery (Configured) bundle" OFF)
if (TEST_RSA_DISCOVERY_CONFIGURED)
    add_celix_container(use_rsa_configured
            BUNDLES
            Celix::rsa_discovery
            Celix::rsa_topology_manager
            my_rsa
            hello
            PROPERTIES
            )
endif ()

option(TEST_RSA_DISCOVERY_ETCD "Test the Discovery (ETCD) bundle" OFF)
if (TEST_RSA_DISCOVERY_ETCD)
    add_celix_container(use_rsa_etcd
            BUNDLES
            Celix::rsa_discovery_etcd
            Celix::rsa_topology_manager
            my_rsa
            hello
            PROPERTIES
            )
endif ()

option(TEST_SHELL "Test shell" OFF)
if (TEST_SHELL)
    add_executable(use_shell test_shell.c)
    target_link_libraries(use_shell PRIVATE Celix::framework Celix::shell_api)
    celix_get_bundle_file(Celix::shell SHELL_BUNDLE)
    target_compile_definitions(use_shell PRIVATE SHELL_BUNDLE_LOCATION="${SHELL_BUNDLE}")
    option(TEST_CXX_SHELL "Test CXX shell" OFF)
    if (TEST_CXX_SHELL)
        add_executable(use_cxx_shell test_cxx_shell.cpp)
        target_link_libraries(use_cxx_shell PRIVATE Celix::framework Celix::shell_api)
        celix_get_bundle_file(Celix::ShellCxx CXX_SHELL_BUNDLE)
        target_compile_definitions(use_cxx_shell PRIVATE CXX_SHELL_BUNDLE_LOCATION="${CXX_SHELL_BUNDLE}")
    endif ()
endif ()

option(TEST_REMOTE_SHELL "Test remote shell" OFF)
if(TEST_REMOTE_SHELL)
    add_celix_container("use_remote_shell" BUNDLES Celix::shell Celix::remote_shell hello)
endif ()

option(TEST_SHELL_TUI "Test shell TUI" OFF)
if(TEST_SHELL_TUI)
    add_celix_container("use_shell_tui" BUNDLES Celix::shell Celix::shell_tui hello)
endif ()

option(TEST_SHELL_WUI "Test shell WUI" OFF)
if(TEST_SHELL_WUI)
    add_celix_container("use_shell_wui" BUNDLES Celix::shell Celix::shell_wui hello)
endif ()

option(TEST_ETCD_LIB "Test ETCD lib" OFF)
if (TEST_ETCD_LIB)
    add_executable(use_etcd_lib test_etcd_lib.c)
    target_link_libraries(use_etcd_lib PRIVATE Celix::etcdlib)
endif ()

option(TEST_LAUNCHER "Test launcher" OFF)
if (TEST_LAUNCHER)
    get_target_property(LAUNCHER_LOCATION Celix::launcher LOCATION)
    add_celix_container("use_launcher"  LAUNCHER ${LAUNCHER_LOCATION} USE_CONFIG  BUNDLES hello)
endif ()

option(TEST_PROMISES "Test promises" OFF)
if (TEST_PROMISES)
    add_executable(use_promises test_promises.cpp)
    target_link_libraries(use_promises PRIVATE Celix::Promises)
endif ()

option(TEST_PUSHSTREAMS "Test pushstreams" OFF)
if (TEST_PUSHSTREAMS)
    add_executable(use_pushstreams test_pushstreams.cpp)
    target_link_libraries(use_pushstreams PRIVATE Celix::PushStreams)
endif ()

