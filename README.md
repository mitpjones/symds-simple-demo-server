
1) 


2) Open 3 terminals and cd to symmetric-server-3.8.32/bin in each one



3) In terminal console 3 create database + demo table with 1 row of data

su - postgres

createdb simpledemodb

psql -U postgres -d simpledemodb -c "CREATE ROLE simpledemo LOGIN PASSWORD 'simpledemo' SUPERUSER INHERIT CREATEDB CREATEROLE NOREPLICATION"

psql -U postgres -d simpledemodb -c "CREATE TABLE demo (id text primary key, text text, active boolean, update_date_time timestamp without time zone NOT NULL, version integer NOT NULL)"

psql -U postgres -d simpledemodb -c "INSERT INTO demo (id, text, active, update_date_time, version) VALUES ('1', 'initial text from server', true, CURRENT_TIMESTAMP, 0)"



4) In terminal console 1 delete staging directory as may be populated from previous run + start Sym DS

cd /home/tim/resources/mash/symmetricDS/symds-simple-demo/symmetric-server-3.8.32/bin

-- delete staging directory
rm -rf /home/tim/resources/mash/symmetricDS/symds-simple-demo/symmetric-server-3.8.32/tmp/simple-demo

rm -rf  /tmp/simple-demo

. ./setenv

./sym

-- should see something like

[startup] - SymmetricWebServer - About to start SymmetricDS web server on host:port 0.0.0.0:31415
[startup] - / - Initializing Spring root WebApplicationContext
[simple-demo] - AbstractSymmetricEngine - Initializing connection to database
[simple-demo] - JdbcDatabasePlatformFactory - Detected database 'PostgreSQL', version '9', protocol 'postgresql'
[simple-demo] - JdbcDatabasePlatformFactory - The IDatabasePlatform being used is org.jumpmind.db.platform.postgresql.PostgreSqlDatabasePlatform
[simple-demo] - PostgreSqlSymmetricDialect - The DbDialect being used is org.jumpmind.symmetric.db.postgresql.PostgreSqlSymmetricDialect
[simple-demo] - ExtensionService - Found 0 extension points from the database that will be registered
[simple-demo] - StagingManager - The staging directory was initialized at the following location: /home/tim/resources/mash/symmetricDS/symds-simple-demo/symmetric-server-3.8.32/tmp/simple-demo
[simple-demo] - ClusterService - This node picked a server id of arch-dadda
[startup] - / - Initializing Spring FrameworkServlet 'rest'
[simple-demo] - ExtensionService - Found 0 extension points from the database that will be registered
[simple-demo] - ClientExtensionService - Found 8 extension points from spring that will be registered
[simple-demo] - AbstractSymmetricEngine - Initializing SymmetricDS database
[simple-demo] - PostgreSqlSymmetricDialect - Checking if SymmetricDS tables need created or altered
[startup] - SymmetricWebServer - Starting JMX HTTP console on port 31416
[startup] - SymmetricWebServer - Joining the web server main thread
HttpAdaptor version 3.0.1 started on port 31416
.
.
.







5) In terminal console 3 insert Sym DS data 

psql -U postgres -d simpledemodb -c "insert into SYM_NODE_GROUP (node_group_id, description) values ('simple-demo-tablet-node-group', 'simple demo tablet client node group')"


psql -U postgres -d simpledemodb -c "INSERT INTO sym_node_group_link VALUES ('simple-demo-server-node-group', 'simple-demo-tablet-node-group', 'W', 1, 0, NULL, NULL, NULL)"
psql -U postgres -d simpledemodb -c "INSERT INTO sym_node_group_link VALUES ('simple-demo-tablet-node-group', 'simple-demo-server-node-group', 'P', 1, 0, NULL, NULL, NULL)"



psql -U postgres -d simpledemodb -c "INSERT INTO sym_router VALUES ('simple-demo-server-to-tablet', NULL, NULL, NULL, 'simple-demo-server-node-group', 'simple-demo-tablet-node-group', NULL, NULL, 1, 1, 1, 1, CURRENT_TIMESTAMP, NULL, CURRENT_TIMESTAMP)"
psql -U postgres -d simpledemodb -c "INSERT INTO sym_router VALUES ('simple-demo-tablet-to-server', NULL, NULL, NULL, 'simple-demo-tablet-node-group', 'simple-demo-server-node-group', NULL, NULL, 1, 1, 1, 1, CURRENT_TIMESTAMP, NULL, CURRENT_TIMESTAMP)"


psql -U postgres -d simpledemodb -c "INSERT INTO sym_channel VALUES ('simple-demo-core', 10, 1000, 10, 100000, 0, 1, 1, 1, 1, 0, 0, 0, 'default', 'default', 'Core demo tables', 'default', 0.000, NULL, NULL, NULL, NULL)"


psql -U postgres -d simpledemodb -c "INSERT INTO sym_trigger (trigger_id, source_catalog_name, source_schema_name, source_table_name, channel_id, reload_channel_id, sync_on_update, sync_on_insert, sync_on_delete, sync_on_incoming_batch, name_for_update_trigger, name_for_insert_trigger, name_for_delete_trigger, sync_on_update_condition, sync_on_insert_condition, sync_on_delete_condition, custom_before_update_text, custom_before_insert_text, custom_before_delete_text, custom_on_update_text, custom_on_insert_text, custom_on_delete_text, external_select, tx_id_expression, channel_expression, excluded_column_names, included_column_names, sync_key_names, use_stream_lobs, use_capture_lobs, use_capture_old_data, use_handle_key_updates, create_time, last_update_by, last_update_time) VALUES ('simple-demo-demo-trig-s-to-t', NULL, NULL, 'demo', 'simple-demo-core', 'simple-demo-core', 1, 1, 1, 1, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, 0, 0, 1, 0, CURRENT_TIMESTAMP, NULL, CURRENT_TIMESTAMP)"
psql -U postgres -d simpledemodb -c "INSERT INTO sym_trigger (trigger_id, source_catalog_name, source_schema_name, source_table_name, channel_id, reload_channel_id, sync_on_update, sync_on_insert, sync_on_delete, sync_on_incoming_batch, name_for_update_trigger, name_for_insert_trigger, name_for_delete_trigger, sync_on_update_condition, sync_on_insert_condition, sync_on_delete_condition, custom_before_update_text, custom_before_insert_text, custom_before_delete_text, custom_on_update_text, custom_on_insert_text, custom_on_delete_text, external_select, tx_id_expression, channel_expression, excluded_column_names, included_column_names, sync_key_names, use_stream_lobs, use_capture_lobs, use_capture_old_data, use_handle_key_updates, create_time, last_update_by, last_update_time) VALUES ('simple-demo-demo-trig-t-to-s', NULL, NULL, 'demo', 'simple-demo-core', 'simple-demo-core', 1, 1, 1, 0, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, 0, 0, 1, 0, CURRENT_TIMESTAMP, NULL, CURRENT_TIMESTAMP)"



psql -U postgres -d simpledemodb -c "INSERT INTO sym_trigger_router VALUES ('simple-demo-demo-trig-s-to-t', 'simple-demo-server-to-tablet', 1, 100, NULL, NULL, 1, 0, CURRENT_TIMESTAMP, NULL, CURRENT_TIMESTAMP)"
psql -U postgres -d simpledemodb -c "INSERT INTO sym_trigger_router VALUES ('simple-demo-demo-trig-t-to-s', 'simple-demo-tablet-to-server', 1, 100, NULL, NULL, 1, 0, CURRENT_TIMESTAMP, NULL, CURRENT_TIMESTAMP)"



psql -U postgres -d simpledemodb -c "INSERT INTO sym_transform_table VALUES ('demo_inbd_on_serv', 'simple-demo-tablet-node-group', 'simple-demo-server-node-group', 'LOAD', NULL, NULL, 'demo', NULL, NULL, 'demo', 1, 'UPD_ROW', 'DEL_ROW', 1, 'IMPLIED', CURRENT_TIMESTAMP, 'Tablet', CURRENT_TIMESTAMP)"
psql -U postgres -d simpledemodb -c "INSERT INTO sym_transform_table VALUES ('demo_inbnd_on_tab', 'simple-demo-server-node-group', 'simple-demo-tablet-node-group', 'LOAD', NULL, NULL, 'demo', NULL, NULL, 'demo', 1, 'UPD_ROW', 'DEL_ROW', 1, 'IMPLIED', CURRENT_TIMESTAMP, 'Tablet', CURRENT_TIMESTAMP)"



psql -U postgres -d simpledemodb -c "INSERT INTO sym_transform_column (transform_id, include_on, target_column_name, source_column_name, pk, transform_type, transform_expression, transform_order, create_time, last_update_by, last_update_time) VALUES ('demo_inbd_on_serv', '*', 'active', 'active', 0, 'bsh', 'if (currentValue != null) {  
            log.info(\"table demo before transform in_active currentValue =<\" + currentValue + \">\"); 
            if (\"Y\".equals(currentValue)) { 
  			     currentValue = \"1\"; 
            } 
            else if (\"N\".equals(currentValue)) { 
                 currentValue = \"0\"; 
            } 
            else { 
                 log.error(\"Error when trying to transform column is_active for table demo, value must be Y or N but was =<\" + currentValue + \">\"); 
            } 
            log.info(\"table demo after transform in_active currentValue =<\" + currentValue + \">\"); 
  	  }  
       return currentValue; ', 1, CURRENT_TIMESTAMP, NULL, CURRENT_TIMESTAMP)"
       
psql -U postgres -d simpledemodb -c "INSERT INTO sym_transform_column (transform_id, include_on, target_column_name, source_column_name, pk, transform_type, transform_expression, transform_order, create_time, last_update_by, last_update_time) VALUES ('demo_inbnd_on_tab', '*', 'active', 'active', 0, 'bsh', 'if (currentValue != null) { 
       		log.info(\"table demo before transform in_active currentValue =<\" + currentValue + \">\"); 
  			if (\"1\".equals(currentValue)) { 
  				currentValue = \"Y\"; 
  			} 
  			else if (\"0\".equals(currentValue)) { 
  				currentValue = \"N\"; 
  			} 
  			else { 
  				log.error(\"Error when trying to transform column is_active for table demo, value must be 1 or 0 but was =<\" + currentValue + \">\"); 
  			} 
  			log.info(\"table demo after transform in_active currentValue =<\" + currentValue + \">\"); 
  		}  
  		return currentValue; ', 1, CURRENT_TIMESTAMP, NULL, CURRENT_TIMESTAMP)"


-- USE_VERSION
psql -U postgres -d simpledemodb -c "INSERT INTO sym_conflict (conflict_id, source_node_group_id, target_node_group_id, target_channel_id, target_catalog_name, target_schema_name, target_table_name, detect_type, detect_expression, resolve_type, ping_back, resolve_changes_only, resolve_row_only, create_time, last_update_by, last_update_time) VALUES ('simple-demo-sym-con-s2t-demo', 'simple-demo-server-node-group', 'simple-demo-tablet-node-group', NULL, NULL, NULL, 'demo', 'USE_VERSION', 'version', 'NEWER_WINS', 'SINGLE_ROW', 1, 1, CURRENT_TIMESTAMP, NULL, CURRENT_TIMESTAMP)"
psql -U postgres -d simpledemodb -c "INSERT INTO sym_conflict (conflict_id, source_node_group_id, target_node_group_id, target_channel_id, target_catalog_name, target_schema_name, target_table_name, detect_type, detect_expression, resolve_type, ping_back, resolve_changes_only, resolve_row_only, create_time, last_update_by, last_update_time) VALUES ('simple-demo-sym-con-t2s-demo', 'simple-demo-tablet-node-group', 'simple-demo-server-node-group', NULL, NULL, NULL, 'demo', 'USE_VERSION', 'version', 'NEWER_WINS', 'SINGLE_ROW', 1, 1, CURRENT_TIMESTAMP, NULL, CURRENT_TIMESTAMP)"



---- USE_TIMESTAMP (possible a bug here, will raise later)
----psql -U postgres -d simpledemodb -c "INSERT INTO sym_conflict (conflict_id, source_node_group_id, target_node_group_id, target_channel_id, target_catalog_name, target_schema_name, target_table_name, detect_type, detect_expression, resolve_type, ping_back, resolve_changes_only, resolve_row_only, create_time, last_update_by, last_update_time) VALUES ('simple-demo-sym-con-s2t-demo', 'simple-demo-server-node-group', 'simple-demo-tablet-node-group', NULL, NULL, NULL, 'demo', 'USE_TIMESTAMP', 'update_date_time', 'NEWER_WINS', 'SINGLE_ROW', 1, 1, CURRENT_TIMESTAMP, NULL, CURRENT_TIMESTAMP)"
----psql -U postgres -d simpledemodb -c "INSERT INTO sym_conflict (conflict_id, source_node_group_id, target_node_group_id, target_channel_id, target_catalog_name, target_schema_name, target_table_name, detect_type, detect_expression, resolve_type, ping_back, resolve_changes_only, resolve_row_only, create_time, last_update_by, last_update_time) VALUES ('simple-demo-sym-con-t2s-demo', 'simple-demo-tablet-node-group', 'simple-demo-server-node-group', NULL, NULL, NULL, 'demo', 'USE_TIMESTAMP', 'update_date_time', 'NEWER_WINS', 'SINGLE_ROW', 1, 1, CURRENT_TIMESTAMP, NULL, CURRENT_TIMESTAMP)"






6) In terminal console 2 open registration for the node group "simple-demo-tablet-node-group" with an external identifier of "simple-demo-server-node"

./symadmin --engine simple-demo open-registration simple-demo-tablet-node-group simple-demo-client-node

-- should se something like ...

Log output will be written to /home/tim/resources/mash/symmetricDS/symds-simple-demo/symmetric-server-3.8.32/logs/symmetric.log
[] - AbstractCommandLauncher - Option: name=engine, value={simple-demo}
[simple-demo] - AbstractSymmetricEngine - Initializing connection to database
[simple-demo] - JdbcDatabasePlatformFactory - Detected database 'PostgreSQL', version '9', protocol 'postgresql'
[simple-demo] - JdbcDatabasePlatformFactory - The IDatabasePlatform being used is org.jumpmind.db.platform.postgresql.PostgreSqlDatabasePlatform
[simple-demo] - PostgreSqlSymmetricDialect - The DbDialect being used is org.jumpmind.symmetric.db.postgresql.PostgreSqlSymmetricDialect
[simple-demo] - ExtensionService - Found 0 extension points from the database that will be registered
[simple-demo] - StagingManager - The staging directory was initialized at the following location: /home/tim/resources/mash/symmetricDS/symds-simple-demo/symmetric-server-3.8.32/tmp/simple-demo
[simple-demo] - ClusterService - This node picked a server id of arch-dadda
[simple-demo] - ExtensionService - Found 0 extension points from the database that will be registered
[simple-demo] - ClientExtensionService - Found 7 extension points from spring that will be registered
[simple-demo] - RegistrationService - Just opened registration for external id of simple-demo-client-node and a node group of simple-demo-tablet-node-group and a node id of simple-demo-client-node
Opened registration for node group of 'simple-demo-tablet-node-group' external ID of 'simple-demo-client-node'



7) In terminal console 1 stop Sym DS and then restart (not sure why necessary)

./sym




8) Start Android app


9) In terminal console 2 run initial load

./symadmin --engine simple-demo reload-node simple-demo-client-node





-----------------------



psql -U postgres -d simpledemodb -c "DROP SCHEMA public CASCADE"
psql -U postgres -d simpledemodb -c "CREATE SCHEMA public"

psql -U postgres -d simpledemodb -c "GRANT ALL ON SCHEMA public TO postgres"
psql -U postgres -d simpledemodb -c "GRANT ALL ON SCHEMA public TO public"



