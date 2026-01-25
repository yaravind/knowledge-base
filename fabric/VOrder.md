# VOrder

- Power BI & SQL engines use Verti-Scan technology + V-Ordered parquet files to achieve in-memore like speed
- **Write time optimization** for Parquet files for faster reads
- Spark and Non Verti-scan engines can also improve speeds by 10%
- V-Order optimizations:
	- Sorting (Adds approxx 15% more to write time but boosts compression by 50%)
	- Row-group distribution
	- Encoding
	- Compression
- Adds 15% to the write time
- Z-Order, compaction, vacuum, time travel, etc. are orthogonal and compatible to V-Order
- **V-Order is disabled by default** for all newly created workspaces to optimize performance for write-heavy data engineering workloads.
- Table properties and optimization commands can be used to control the V-Order of its partitions.
	- `spark.sql.parquet.vorder.default`: Controls **session-level** V-Order writing. Set to false by default in new Fabric workspaces.
	- `TBLPROPERTIES("delta.parquet.vorder.enabled")`: Controls default V-Order behavior at the table level.
	- `parquet.vorder.enabled`: Used to control V-Order at the write operation level.
