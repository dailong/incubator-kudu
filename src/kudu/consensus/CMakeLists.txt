# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

#########################################
# consensus_metadata_proto
#########################################

PROTOBUF_GENERATE_CPP(
  METADATA_PROTO_SRCS METADATA_PROTO_HDRS METADATA_PROTO_TGTS
  SOURCE_ROOT ${CMAKE_CURRENT_SOURCE_DIR}/../..
  BINARY_ROOT ${CMAKE_CURRENT_BINARY_DIR}/../..
  PROTO_FILES
    metadata.proto
    opid.proto)
set(METADATA_PROTO_LIBS
  kudu_common_proto
  fs_proto
  protobuf)
ADD_EXPORTABLE_LIBRARY(consensus_metadata_proto
  SRCS ${METADATA_PROTO_SRCS}
  DEPS ${METADATA_PROTO_LIBS}
  NONLINK_DEPS ${METADATA_PROTO_TGTS})

#########################################
# consensus_proto
#########################################

KRPC_GENERATE(
  CONSENSUS_KRPC_SRCS CONSENSUS_KRPC_HDRS CONSENSUS_KRPC_TGTS
  SOURCE_ROOT ${CMAKE_CURRENT_SOURCE_DIR}/../..
  BINARY_ROOT ${CMAKE_CURRENT_BINARY_DIR}/../..
  PROTO_FILES consensus.proto)
list(APPEND CONSENSUS_KRPC_SRCS opid_util.cc)
set(CONSENSUS_KRPC_LIBS
  cfile_proto
  consensus_metadata_proto
  krpc
  kudu_common_proto
  rpc_header_proto
  protobuf
  tablet_proto
  tserver_admin_proto
  wire_protocol_proto)

ADD_EXPORTABLE_LIBRARY(consensus_proto
  SRCS ${CONSENSUS_KRPC_SRCS}
  DEPS ${CONSENSUS_KRPC_LIBS}
  NONLINK_DEPS ${CONSENSUS_KRPC_TGTS})

#########################################
# log_proto
#########################################

PROTOBUF_GENERATE_CPP(
  LOG_PROTO_SRCS LOG_PROTO_HDRS LOG_PROTO_TGTS
  SOURCE_ROOT ${CMAKE_CURRENT_SOURCE_DIR}/../..
  BINARY_ROOT ${CMAKE_CURRENT_BINARY_DIR}/../..
  PROTO_FILES log.proto)

add_library(log_proto ${LOG_PROTO_SRCS} ${LOG_PROTO_HDRS})
target_link_libraries(log_proto
  consensus_proto)

set(LOG_SRCS
  log_util.cc
  log.cc
  log_anchor_registry.cc
  log_index.cc
  log_reader.cc
  log_metrics.cc
)

add_library(log ${LOG_SRCS})
target_link_libraries(log
  server_common
  gutil
  kudu_common
  kudu_fs
  consensus_proto
  log_proto
  consensus_metadata_proto)

set(CONSENSUS_SRCS
  consensus.cc
  consensus_meta.cc
  consensus_peers.cc
  consensus_queue.cc
  leader_election.cc
  local_consensus.cc
  log_cache.cc
  peer_manager.cc
  quorum_util.cc
  raft_consensus.cc
  raft_consensus_state.cc
)

add_library(consensus ${CONSENSUS_SRCS})
target_link_libraries(consensus
  consensus_proto
  kudu_common
  log
  protobuf)

set(KUDU_TEST_LINK_LIBS
  log
  consensus
  tserver
  cfile
  tablet
  kudu_util
  ${KUDU_MIN_TEST_LIBS}
)

ADD_KUDU_TEST(consensus_meta-test)
ADD_KUDU_TEST(consensus_peers-test)
ADD_KUDU_TEST(consensus_queue-test)
ADD_KUDU_TEST(leader_election-test)
ADD_KUDU_TEST(log-test)
ADD_KUDU_TEST(log_anchor_registry-test)
ADD_KUDU_TEST(log_cache-test)
ADD_KUDU_TEST(log_index-test)
ADD_KUDU_TEST(mt-log-test)
ADD_KUDU_TEST(quorum_util-test)
ADD_KUDU_TEST(raft_consensus_quorum-test)
ADD_KUDU_TEST(raft_consensus_state-test)
ADD_KUDU_TEST(raft_consensus-test)

#Tools
add_executable(log-dump log-dump.cc)
target_link_libraries(log-dump
  log
  ${KUDU_BASE_LIBS})
