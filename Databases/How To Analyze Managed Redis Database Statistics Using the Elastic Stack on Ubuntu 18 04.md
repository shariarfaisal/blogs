# How To Analyze Managed Redis Database Statistics Using the Elastic Stack on Ubuntu 18 04

```Databases``` ```Ubuntu``` ```Elasticsearch``` ```Redis``` ```Monitoring```

The author selected the Free and Open Source Fund to receive a donation as part of the Write for DOnations program.


## Introduction


Database monitoring is the continuous process of systematically tracking various metrics that show how the database is performing. By observing performance data, you can gain valuable insights and identify possible bottlenecks, as well as find additional ways of improving database performance. Such systems often implement alerting that notifies administrators when things go wrong. Gathered statistics can be used to not only improve the configuration and workflow of the database, but also those of client applications.


The benefit of using the Elastic Stack (ELK stack) for monitoring your managed database is its excellent support for searching and the ability to ingest new data very quickly. It does not excel at updating the data, but this trade-off is acceptable for monitoring and logging purposes, where past data is almost never changed. Elasticsearch offers a powerful means of querying the data, which you can use through Kibana to get a better understanding of how the database fares through different time periods. This will allow you to correlate database load with real-life events to gain insight into how the database is being used.


In this tutorial, you’ll import database metrics, generated by the Redis INFO command, into Elasticsearch via Logstash. This entails configuring Logstash to periodically run the command, parse its output, and send it to Elasticsearch for indexing immediately afterward. The imported data can later be analyzed and visualized in Kibana. By the end of the tutorial, you’ll have an automated system pulling in Redis statistics for later analysis.


# Prerequisites


- An Ubuntu 18.04 server with at least 8 GB RAM, root privileges, and a secondary, non-root account. You can set this up by following this initial server setup guide. For this tutorial, the non-root user is sammy.
- Java 8 installed on your server. For installation instructions, visit How To Install Java with apt on Ubuntu 18.04 and follow the commands outlined in the first step. You don’t need to install the Java Development Kit (JDK).
- Nginx installed on your server. For a guide on how to do that, see the How To Install Nginx on Ubuntu 18.04 tutorial.
- Elasticsearch and Kibana installed on your server. Complete the first two steps of the How To Install Elasticsearch, Logstash, and Kibana (Elastic Stack) on Ubuntu 18.04 tutorial.
- A Redis managed database provisioned from DigitalOcean with connection information available. Make sure that your server’s IP address is on the whitelist. For a guide on creating a Redis database using the DigitalOcean Control Panel, visit the Redis Quickstart guide.
- Redli installed on your server according to the How To Connect to a Managed Database on Ubuntu 18.04 tutorial.

# Step 1 — Installing and Configuring Logstash


In this section, you will install Logstash and configure it to pull statistics from your Redis database cluster, then parse them to send to Elasticsearch for indexing.


Start off by installing Logstash with the following command:


```
sudo apt install logstash -y


```


Once Logstash is installed, enable the service to automatically start on boot:


```
sudo systemctl enable logstash


```


Before configuring Logstash to pull the statistics, let’s see what the data itself looks like. To connect to your Redis database, head over to your Managed Database Control Panel, and under the Connection details panel, select Flags from the dropdown:





You’ll be shown a preconfigured command for the Redli client, which you’ll use to connect to your database. Click Copy and run the following command on your server, replacing redli_flags_command with the command you have just copied:


```
redli_flags_command info


```


Since the output from this command is long, we’ll explain this broken down into its different sections.


In the output of the Redis info command, sections are marked with #, which signifies a comment. The values are populated in the form of key:value, which makes them relatively easy to parse.


The Server section contains technical information about the Redis build, such as its version and the Git commit it’s based on, while the Clients section provides the number of currently opened connections.


```
Output# Server
redis_version:6.2.6
redis_git_sha1:4f4e829a
redis_git_dirty:1
redis_build_id:5861572cb79aebf3
redis_mode:standalone
os:Linux 5.11.12-300.fc34.x86_64 x86_64
arch_bits:64
multiplexing_api:epoll
atomicvar_api:atomic-builtin
gcc_version:11.2.1
process_id:79
process_supervised:systemd
run_id:b8a0aa25d8f49a879112a04a817ac2acd92e0c75
tcp_port:25060
server_time_usec:1640878632737564
uptime_in_seconds:1679
uptime_in_days:0
hz:10
configured_hz:10
lru_clock:13488680
executable:/usr/bin/redis-server
config_file:/etc/redis.conf
io_threads_active:0

# Clients
connected_clients:4
cluster_connections:0
maxclients:10032
client_recent_max_input_buffer:24
client_recent_max_output_buffer:0
...

```


Memory confirms how much RAM Redis has allocated for itself, as well as the maximum amount of memory it can possibly use. If it starts running out of memory, it will free up keys using the strategy you specified in the Control Panel (shown in the maxmemory_policy field in this output).


```
Output...
# Memory
used_memory:977696
used_memory_human:954.78K
used_memory_rss:9977856
used_memory_rss_human:9.52M
used_memory_peak:977696
used_memory_peak_human:954.78K
used_memory_peak_perc:100.00%
used_memory_overhead:871632
used_memory_startup:810128
used_memory_dataset:106064
used_memory_dataset_perc:63.30%
allocator_allocated:947216
allocator_active:1273856
allocator_resident:3510272
total_system_memory:1017667584
total_system_memory_human:970.52M
used_memory_lua:37888
used_memory_lua_human:37.00K
used_memory_scripts:0
used_memory_scripts_human:0B
number_of_cached_scripts:0
maxmemory:455081984
maxmemory_human:434.00M
maxmemory_policy:noeviction
allocator_frag_ratio:1.34
allocator_frag_bytes:326640
allocator_rss_ratio:2.76
allocator_rss_bytes:2236416
rss_overhead_ratio:2.84
rss_overhead_bytes:6467584
mem_fragmentation_ratio:11.43
mem_fragmentation_bytes:9104832
mem_not_counted_for_evict:0
mem_replication_backlog:0
mem_clients_slaves:0
mem_clients_normal:61504
mem_aof_buffer:0
mem_allocator:jemalloc-5.1.0
active_defrag_running:0
lazyfree_pending_objects:0
...

```


In the Persistence section, you can see the last time Redis saved the keys it stores to disk, and if it was successful. The Stats section provides numbers related to client and in-cluster connections, the number of times the requested key was (or wasn’t) found, and so on.


```
Output...
# Persistence
loading:0
current_cow_size:0
current_cow_size_age:0
current_fork_perc:0.00
current_save_keys_processed:0
current_save_keys_total:0
rdb_changes_since_last_save:0
rdb_bgsave_in_progress:0
rdb_last_save_time:1640876954
rdb_last_bgsave_status:ok
rdb_last_bgsave_time_sec:1
rdb_current_bgsave_time_sec:-1
rdb_last_cow_size:217088
aof_enabled:0
aof_rewrite_in_progress:0
aof_rewrite_scheduled:0
aof_last_rewrite_time_sec:-1
aof_current_rewrite_time_sec:-1
aof_last_bgrewrite_status:ok
aof_last_write_status:ok
aof_last_cow_size:0
module_fork_in_progress:0
module_fork_last_cow_size:0

# Stats
total_connections_received:202
total_commands_processed:2290
instantaneous_ops_per_sec:0
total_net_input_bytes:38034
total_net_output_bytes:1103968
instantaneous_input_kbps:0.01
instantaneous_output_kbps:0.00
rejected_connections:0
sync_full:0
sync_partial_ok:0
sync_partial_err:0
expired_keys:0
expired_stale_perc:0.00
expired_time_cap_reached_count:0
expire_cycle_cpu_milliseconds:29
evicted_keys:0
keyspace_hits:0
keyspace_misses:0
pubsub_channels:0
pubsub_patterns:0
latest_fork_usec:452
total_forks:1
migrate_cached_sockets:0
slave_expires_tracked_keys:0
active_defrag_hits:0
active_defrag_misses:0
active_defrag_key_hits:0
active_defrag_key_misses:0
tracking_total_keys:0
tracking_total_items:0
tracking_total_prefixes:0
unexpected_error_replies:0
total_error_replies:0
dump_payload_sanitizations:0
total_reads_processed:2489
total_writes_processed:2290
io_threaded_reads_processed:0
io_threaded_writes_processed:0
...

```


By looking at the role under Replication, you’ll know if you’re connected to a primary or replica node. The rest of the section provides the number of currently connected replicas and the amount of data that the replica is lacking in regards to the primary. There may be additional fields if the instance you are connected to is a replica.



Note: The Redis project uses the terms “master” and “slave” in its documentation and in various commands. DigitalOcean generally prefers the alternative terms “primary” and “replica.” This guide will default to the terms “primary” and “replica” whenever possible, but note that there are a few instances where the terms “master” and “slave” unavoidably come up.

```
Output...
# Replication
role:master
connected_slaves:0
master_failover_state:no-failover
master_replid:f727fad3691f2a8d8e593b087c468bbb83703af3
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:0
second_repl_offset:-1
repl_backlog_active:0
repl_backlog_size:45088768
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0
...

```


Under CPU, you’ll see the amount of system (used_cpu_sys) and user (used_cpu_user) CPU power Redis is consuming at the moment. The Cluster section contains only one unique field, cluster_enabled, which serves to indicate that the Redis cluster is running.


```
Output...
# CPU
used_cpu_sys:1.617986
used_cpu_user:1.248422
used_cpu_sys_children:0.000000
used_cpu_user_children:0.001459
used_cpu_sys_main_thread:1.567638
used_cpu_user_main_thread:1.218768

# Modules

# Errorstats

# Cluster
cluster_enabled:0

# Keyspace


```


Logstash will be tasked to periodically run the info command on your Redis database (similar to how you just did), parse the results, and send them to Elasticsearch. You’ll then be able to access them later from Kibana.


You’ll store the configuration for indexing Redis statistics in Elasticsearch in a file named redis.conf under the /etc/logstash/conf.d directory, where Logstash stores configuration files. When started as a service, it will automatically run them in the background.


Create redis.conf using your favorite editor (for example, nano):


```
sudo nano /etc/logstash/conf.d/redis.conf


```


Add the following lines:


/etc/logstash/conf.d/redis.conf
```
input {
	exec {
		command => "redis_flags_command info"
		interval => 10
		type => "redis_info"
	}
}

filter {
	kv {
		value_split => ":"
		field_split => "\r\n"
		remove_field => [ "command", "message" ]
	}

	ruby {
		code =>
		"
		event.to_hash.keys.each { |k|
			if event.get(k).to_i.to_s == event.get(k) # is integer?
				event.set(k, event.get(k).to_i) # convert to integer
			end
			if event.get(k).to_f.to_s == event.get(k) # is float?
				event.set(k, event.get(k).to_f) # convert to float
			end
		}
		puts 'Ruby filter finished'
		"
	}
}

output {
    elasticsearch {
        hosts => "http://localhost:9200"
        index => "%{type}"
    }
}

```


Remember to replace redis_flags_command with the command shown in the control panel that you used earlier in the step.


You define an input, which is a set of filters that will run on the collected data, and an output that will send the filtered data to Elasticsearch. The input consists of the exec command, which will run a command on the server periodically, after a set time interval (expressed in seconds). It also specifies a type parameter that defines the document type when indexed in Elasticsearch. The exec block passes down an object containing two fields, command and message string. The command field will contain the command that was run, and the message will contain its output.


There are two filters that will run sequentially on the data collected from the input. The kv filter stands for key-value filter, and is built-in to Logstash. It is used for parsing data in the general form of keyvalue_separatorvalue and provides parameters for specifying what are considered value and field separators. The field separator pertains to strings that separate the data formatted in the general form from each other. In the case of the output of the Redis INFO command, the field separator (field_split) is a new line, and the value separator (value_split) is :. Lines that do not follow the defined form will be discarded, including comments.


To configure the kv filter, you pass : to thevalue_split parameter, and \r\n (signifying a new line) to the field_split parameter. You also order it to remove the command and message fields from the current data object by passing them to remove_field as elements of an array, because they contain data that are now useless.


The kv filter represents the value it parsed as a string (text) type by design. This raises an issue because Kibana can’t easily process string types, even if it’s actually a number. To solve this, you’ll use custom Ruby code to convert the number-only strings to numbers, where possible. The second filter is a ruby block that provides a code parameter accepting a string containing the code to be run.


event is a variable that Logstash provides to your code, and contains the current data in the filter pipeline. As was noted before, filters run one after another, meaning that the Ruby filter will receive the parsed data from the kv filter. The Ruby code itself converts the event to a Hash and traverses through the keys, then checks if the value associated with the key could be represented as an integer or as a float (a number with decimals). If it can, the string value is replaced with the parsed number. When the loop finishes, it prints out a message (Ruby filter finished) to report progress.


The output sends the processed data to Elasticsearch for indexing. The resulting document will be stored in the redis_info index, defined in the input and passed in as a parameter to the output block.


Save and close the file.


You’ve installed Logstash using apt and configured it to periodically request statistics from Redis, process them, and send them to your Elasticsearch instance.


# Step 2 — Testing the Logstash Configuration


Now you’ll test the configuration by running Logstash to verify it will properly pull the data.


Logstash supports running a specific configuration by passing its file path to the -f parameter. Run the following command to test your new configuration from the last step:


```
sudo /usr/share/logstash/bin/logstash -f /etc/logstash/conf.d/redis.conf


```


It may take some time to show the output, but you’ll soon see something similar to the following:


```
OutputUsing bundled JDK: /usr/share/logstash/jdk
OpenJDK 64-Bit Server VM warning: Option UseConcMarkSweepGC was deprecated in version 9.0 and will likely be removed in a future release.
WARNING: Could not find logstash.yml which is typically located in $LS_HOME/config or /etc/logstash. You can specify the path using --path.settings. Continuing using the defaults
Could not find log4j2 configuration at path /usr/share/logstash/config/log4j2.properties. Using default config which logs errors to the console
[INFO ] 2021-12-30 15:42:08.887 [main] runner - Starting Logstash {"logstash.version"=>"7.16.2", "jruby.version"=>"jruby 9.2.20.1 (2.5.8) 2021-11-30 2a2962fbd1 OpenJDK 64-Bit Server VM 11.0.13+8 on 11.0.13+8 +indy +jit [linux-x86_64]"}
[INFO ] 2021-12-30 15:42:08.932 [main] settings - Creating directory {:setting=>"path.queue", :path=>"/usr/share/logstash/data/queue"}
[INFO ] 2021-12-30 15:42:08.939 [main] settings - Creating directory {:setting=>"path.dead_letter_queue", :path=>"/usr/share/logstash/data/dead_letter_queue"}
[WARN ] 2021-12-30 15:42:09.406 [LogStash::Runner] multilocal - Ignoring the 'pipelines.yml' file because modules or command line options are specified
[INFO ] 2021-12-30 15:42:09.449 [LogStash::Runner] agent - No persistent UUID file found. Generating new UUID {:uuid=>"acc4c891-936b-4271-95de-7d41f4a41166", :path=>"/usr/share/logstash/data/uuid"}
[INFO ] 2021-12-30 15:42:10.985 [Api Webserver] agent - Successfully started Logstash API endpoint {:port=>9600, :ssl_enabled=>false}
[INFO ] 2021-12-30 15:42:11.601 [Converge PipelineAction::Create<main>] Reflections - Reflections took 77 ms to scan 1 urls, producing 119 keys and 417 values
[WARN ] 2021-12-30 15:42:12.215 [Converge PipelineAction::Create<main>] plain - Relying on default value of `pipeline.ecs_compatibility`, which may change in a future major release of Logstash. To avoid unexpected changes when upgrading Logstash, please explicitly declare your desired ECS Compatibility mode.
[WARN ] 2021-12-30 15:42:12.366 [Converge PipelineAction::Create<main>] plain - Relying on default value of `pipeline.ecs_compatibility`, which may change in a future major release of Logstash. To avoid unexpected changes when upgrading Logstash, please explicitly declare your desired ECS Compatibility mode.
[WARN ] 2021-12-30 15:42:12.431 [Converge PipelineAction::Create<main>] elasticsearch - Relying on default value of `pipeline.ecs_compatibility`, which may change in a future major release of Logstash. To avoid unexpected changes when upgrading Logstash, please explicitly declare your desired ECS Compatibility mode.
[INFO ] 2021-12-30 15:42:12.494 [[main]-pipeline-manager] elasticsearch - New Elasticsearch output {:class=>"LogStash::Outputs::ElasticSearch", :hosts=>["http://localhost:9200"]}
[INFO ] 2021-12-30 15:42:12.755 [[main]-pipeline-manager] elasticsearch - Elasticsearch pool URLs updated {:changes=>{:removed=>[], :added=>[http://localhost:9200/]}}
[WARN ] 2021-12-30 15:42:12.955 [[main]-pipeline-manager] elasticsearch - Restored connection to ES instance {:url=>"http://localhost:9200/"}
[INFO ] 2021-12-30 15:42:12.967 [[main]-pipeline-manager] elasticsearch - Elasticsearch version determined (7.16.2) {:es_version=>7}
[WARN ] 2021-12-30 15:42:12.968 [[main]-pipeline-manager] elasticsearch - Detected a 6.x and above cluster: the `type` event field won't be used to determine the document _type {:es_version=>7}
[WARN ] 2021-12-30 15:42:13.065 [[main]-pipeline-manager] kv - Relying on default value of `pipeline.ecs_compatibility`, which may change in a future major release of Logstash. To avoid unexpected changes when upgrading Logstash, please explicitly declare your desired ECS Compatibility mode.
[INFO ] 2021-12-30 15:42:13.090 [Ruby-0-Thread-10: :1] elasticsearch - Using a default mapping template {:es_version=>7, :ecs_compatibility=>:disabled}
[INFO ] 2021-12-30 15:42:13.147 [Ruby-0-Thread-10: :1] elasticsearch - Installing Elasticsearch template {:name=>"logstash"}
[INFO ] 2021-12-30 15:42:13.192 [[main]-pipeline-manager] javapipeline - Starting pipeline {:pipeline_id=>"main", "pipeline.workers"=>2, "pipeline.batch.size"=>125, "pipeline.batch.delay"=>50, "pipeline.max_inflight"=>250, "pipeline.sources"=>["/etc/logstash/conf.d/redis.conf"], :thread=>"#<Thread:0x5104e975 run>"}
[INFO ] 2021-12-30 15:42:13.973 [[main]-pipeline-manager] javapipeline - Pipeline Java execution initialization time {"seconds"=>0.78}
[INFO ] 2021-12-30 15:42:13.983 [[main]-pipeline-manager] exec - Registering Exec Input {:type=>"redis_info", :command=>"redli --tls -h db-redis-fra1-68603-do-user-1446234-0.b.db.ondigitalocean.com -a hnpJxAgoH3Om3UwM -p 25061 info", :interval=>10, :schedule=>nil}
[INFO ] 2021-12-30 15:42:13.994 [[main]-pipeline-manager] javapipeline - Pipeline started {"pipeline.id"=>"main"}
[INFO ] 2021-12-30 15:42:14.034 [Agent thread] agent - Pipelines running {:count=>1, :running_pipelines=>[:main], :non_running_pipelines=>[]}
Ruby filter finished
Ruby filter finished
Ruby filter finished
...

```


You’ll see the Ruby filter finished message being printed at regular intervals (set to 10 seconds in the previous step), which means that the statistics are being shipped to Elasticsearch.


You can exit Logstash by clicking CTRL + C on your keyboard. As previously mentioned, Logstash will automatically run all config files found under /etc/logstash/conf.d in the background when started as a service. Run the following command to start it:


```
sudo systemctl start logstash


```


You’ve run Logstash to check if it can connect to your Redis cluster and gather data. Next, you’ll explore some of the statistical data in Kibana.


# Step 3 — Exploring Imported Data in Kibana


In this section, you’ll explore and visualize the statistical data describing your database’s performance in Kibana.


In your web browser, navigate to your domain where you exposed Kibana as a part of the prerequisites. You’ll see the default welcome page:





Before exploring the data Logstash is sending to Elasticsearch, you’ll first need to add the redis_info index to Kibana. To do so, first select Explore on my own from the welcome page and then open the hamburger menu in the upper left corner. Under Analytics, click on Discover.





Kibana will then prompt you to create a new index pattern:





Press on Create index pattern. You’ll see a form for creating a new Index Pattern. Index Patterns in Kibana provide a way to pull in data from multiple Elasticsearch indexes at once, and can be used to explore only one index.





On the right, Kibana lists all available indexes, such as redis_info that you’ve configured Logstash to use. Type it in the Name text field and select @timestamp from the dropdown as the Timestamp field. When you’re done, press on the Create index pattern button below.


To create and see existing visualizations, open the hamburger menu. Under Analytics, select Dashboard. When it loads, press on Create visualization to start creating a new one:





The left-side panel provides a list of values that Kibana can use to draw the visualization, which will be shown on the central part of the screen. On the upper-right hand side of the screen is the date range picker. If the @timestamp field is being used in the visualization, Kibana will only show the data belonging to the time interval specified in the range picker.


From the dropdown in the main part of the page, select Line under the Line and area section. Then, find the used_memory field from the list on the left and drag it to the central part. You’ll soon see a line visualization of the median amount of used memory through time:





On the right side, you can configure how the horizontal and vertical axis are processed. There, you can set the vertical axis to show the average values instead of median by pressing on the shown axis:





You can select a different function, or supply your own:





The graph will be immediately refreshed with the updated values.


In this step, you have visualized memory usage of your managed Redis database using Kibana. This will allow you to gain a better understanding of how your database is being used, which will help you optimize client applications, as well as your database itself.


# Conclusion


You now have the Elastic stack installed on your server and configured to pull statistics data from your managed Redis database on a regular basis. You can analyze and visualize the data using Kibana, or some other suitable software, which will help you gather valuable insights into and real-world correlations about how your database is performing.


For more information about what you can do with your Redis Managed Database, visit the product docs. If you’d like to present the database statistics using another visualization type, check out the Kibana docs for further instructions.


