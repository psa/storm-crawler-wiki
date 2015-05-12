###Registering Metadata for Kryo Serialization
If your Storm-Crawler topology doesn't extend `com.digitalpebble.storm.crawler.ConfigurableTopology`, you will need to manually register Storm-Crawler's `Metadata` class for serialization in Storm. For more information on Kryo serialization in Apache Storm, you can refer to the [documentation](https://storm.apache.org/documentation/Serialization.html).

To register `Metadata` for serialization, you'll need to import `backtype.storm.Config` and `com.digitalpebble.storm.crawler.Metadata`. Then, in your topology class, you'll register the class with:

```java
Config.registerSerialization(conf, Metadata.class);
```
where `conf` is your Storm configuration for the topology.

Because Storm topology main classes can take many forms, we've provided a more complete example below (based on storm-starter's [ExclamationTopology](https://github.com/apache/storm/blob/847958cad438766cb7f39ce649fe7a3506b61b3a/examples/storm-starter/src/jvm/storm/starter/ExclamationTopology.java).

```java
package storm.starter;

import backtype.storm.Config;
import backtype.storm.LocalCluster;
import backtype.storm.StormSubmitter;
import backtype.storm.task.OutputCollector;
import backtype.storm.task.TopologyContext;
import backtype.storm.testing.TestWordSpout;
import backtype.storm.topology.OutputFieldsDeclarer;
import backtype.storm.topology.TopologyBuilder;
import backtype.storm.topology.base.BaseRichBolt;
import backtype.storm.tuple.Fields;
import backtype.storm.tuple.Tuple;
import backtype.storm.tuple.Values;
import backtype.storm.utils.Utils;

import com.digitalpebble.storm.crawler.Metadata;

import java.util.Map;

/**
 * This is a basic example of a Storm topology, with serialization for storm-crawler's Metadata.
 */
public class ExclamationTopology {

  public static class ExclamationBolt extends BaseRichBolt {
    OutputCollector _collector;

    @Override
    public void prepare(Map conf, TopologyContext context, OutputCollector collector) {
      _collector = collector;
    }

    @Override
    public void execute(Tuple tuple) {
      _collector.emit(tuple, new Values(tuple.getString(0) + "!!!"));
      _collector.ack(tuple);
    }

    @Override
    public void declareOutputFields(OutputFieldsDeclarer declarer) {
      declarer.declare(new Fields("word"));
    }


  }

  public static void main(String[] args) throws Exception {
    TopologyBuilder builder = new TopologyBuilder();

    builder.setSpout("word", new TestWordSpout(), 10);
    builder.setBolt("exclaim1", new ExclamationBolt(), 3).shuffleGrouping("word");
    builder.setBolt("exclaim2", new ExclamationBolt(), 2).shuffleGrouping("exclaim1");

    Config conf = new Config();
    conf.setDebug(true);

    // Register Metadata for serialization
    Config.registerSerialization(conf, Metadata.class);

    if (args != null && args.length > 0) {
      conf.setNumWorkers(3);

      StormSubmitter.submitTopologyWithProgressBar(args[0], conf, builder.createTopology());
    }
    else {

      LocalCluster cluster = new LocalCluster();
      cluster.submitTopology("test", conf, builder.createTopology());
      Utils.sleep(10000);
      cluster.killTopology("test");
      cluster.shutdown();
    }
  }
}
```