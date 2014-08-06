SET(OTTERYSRC            chacha_merged.c
                         ottery.c
                         ottery_cpuinfo.c
                         ottery_entropy.c
                         ottery_global.c)

ADD_LIBRARY(ottery SHARED ${OTTERYSRC})
install(TARGETS ottery
     LIBRARY DESTINATION lib COMPONENT libraries
     ARCHIVE DESTINATION lib/static COMPONENT libraries)
