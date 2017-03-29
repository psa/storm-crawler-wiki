Assuming you have a storm-crawler setup all ready to use (could be generated from the archetype for instance) you can debug it by either:
* run the Java topology from Eclipse in debug mode
* `export STORM_JAR_JVM_OPTS="-agentlib:jdwp=transport=dt_socket,server=y,suspend=y,address=localhost:8000"` then run the topology in the usual way with `storm jar ... ` then remote debug from Eclipse

In both cases the topology will need to run in local mode, i.e not deployed to a Storm cluster.