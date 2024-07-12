set(SOURCES main.cpp)

add_executable(dpdk_example ${SOURCES})

# Линковка с библиотеками DPDK и системными библиотеками
target_link_libraries(dpdk_example
    ${DPDK_EAL}
    ${DPDK_MEMPOOL}
    ${DPDK_RING}
    ${DPDK_MBUF}
    ${DPDK_ETHDEV}
    ${LIB_DL}
    ${LIB_NUMA}
    ${LIB_PTHREAD}
)
