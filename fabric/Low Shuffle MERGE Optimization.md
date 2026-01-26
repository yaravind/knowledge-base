# Low Shuffle MERGE Optimization

- The current algorithm in the open source distribution of Delta Lake **isn't fully optimized for handling unmodified rows.** 
- The Microsoft Spark Delta team implemented a custom Low Shuffle Merge optimization, **unmodified rows are excluded from an expensive shuffling operation **that is needed for updating matched rows.
- The implementation is controlled by the `spark.microsoft.delta.merge.lowShuffle.enabled`

Reference: https://learn.microsoft.com/en-us/azure/synapse-analytics/spark/low-shuffle-merge-for-apache-spark
