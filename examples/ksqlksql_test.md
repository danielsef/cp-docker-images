


docker run --net=host --rm confluentinc/cp-kafka:latest kafka-topics --create --topic foo --partitions 1 --replication-factor 1 --if-not-exists --zookeeper localhost:32181

docker run --net=host --rm confluentinc/cp-kafka:latest kafka-topics --describe --topic foo --zookeeper localhost:32181

docker run --net=host --rm confluentinc/cp-kafka:latest bash -c "seq 42 | kafka-console-producer --broker-list localhost:29092 --topic foo && echo 'Produced 42 messages.'"

docker run --net=host --rm confluentinc/cp-kafka:latest kafka-console-consumer --bootstrap-server localhost:29092 --topic foo --from-beginning --max-messages 42



-------------------------------------
https://docs.confluent.io/current/ksql/docs/tutorials/basics-docker.html#ksql-quickstart-docker
-------------------------------------

docker run -it confluentinc/cp-ksql-cli http://192.168.0.143:8088

--------------------------------------

docker run --rm --name datagen-pageviews \
  confluentinc/ksql-examples:5.1.0 \
  ksql-datagen \
      bootstrap-server=192.168.0.143:29092 \
      quickstart=pageviews \
      format=delimited \
      topic=pageviews \
      maxInterval=500


docker run --rm --name datagen-users \
  confluentinc/ksql-examples:5.1.0 \
  ksql-datagen \
      bootstrap-server=192.168.0.143:29092 \
      quickstart=users \
      format=json \
      topic=users \


PRINT 'users';
PRINT 'pageviews';

CREATE STREAM pageviews_original (viewtime bigint, userid varchar, pageid varchar) WITH \
(kafka_topic='pageviews', value_format='DELIMITED');




CREATE TABLE users_original (registertime BIGINT, gender VARCHAR, regionid VARCHAR, userid VARCHAR) WITH \
(kafka_topic='users', value_format='JSON', key = 'userid');


SELECT pageid FROM pageviews_original LIMIT 3;


CREATE STREAM pageviews_enriched AS \
SELECT users_original.userid AS userid, pageid, regionid, gender \
FROM pageviews_original \
LEFT JOIN users_original \
  ON pageviews_original.userid = users_original.userid;



SELECT * FROM pageviews_enriched;


CREATE STREAM pageviews_female AS \
SELECT * FROM pageviews_enriched \
WHERE gender = 'FEMALE';

CREATE STREAM pageviews_female_like_89 \
  WITH (kafka_topic='pageviews_enriched_r8_r9') AS \
SELECT * FROM pageviews_female \
WHERE regionid LIKE '%_8' OR regionid LIKE '%_9';



!!! Error !!!
CREATE TABLE pageviews_regions \
  WITH (VALUE_FORMAT='avro') AS \
SELECT gender, regionid , COUNT(*) AS numusers \
FROM pageviews_enriched \
  WINDOW TUMBLING (size 30 second) \
GROUP BY gender, regionid \
HAVING COUNT(*) > 1;


DESCRIBE pageviews_regions;


SHOW QUERIES;











