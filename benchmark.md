# Some benchmarks for Kafka consumers and producers

How to run these commands:
- Download Kafka binaries from https://kafka.apache.org/downloads, and run the commands from bin directory. (You will need Java to be already setup)
- You can run the commands from inside one of the 3 containers. (Not recommanded for testing performance)

## Create a new topic:
```
kafka-topics.sh --bootstrap-server kafka-0:19092 --topic test_topic --create --partitions 16 --replication-factor 2 --config min.insync.replicas=2
```
<br>

## Producer Unoptimized Performance Test
```
time ./kafka-producer-perf-test.sh \
--topic test_topic \
--throughput -1 \
--num-records 3000000 \
--record-size 1024 \
--producer-props acks=all \
bootstrap.servers=kafka-0:19092,kafka-1:29092,kafka-2:39092 --print-metrics | grep \
"3000000 records sent\|\
producer-metrics:outgoing-byte-rate\|\
producer-metrics:bufferpool-wait-ratio\|\
producer-metrics:record-queue-time-avg\|\
producer-metrics:request-latency-avg\|\
producer-metrics:batch-size-avg"

3000000 records sent, 110595.001106 records/sec (108.00 MB/sec), 272.11 ms avg latency, 395.00 ms max latency, 274 ms 50th, 325 ms 95th, 346 ms 99th, 364 ms 99.9th.
producer-metrics:batch-size-avg:{client-id=perf-producer-client}                             : 15556.000
producer-metrics:bufferpool-wait-ratio:{client-id=perf-producer-client}                      : 0.243
producer-metrics:outgoing-byte-rate:{client-id=perf-producer-client}                         : 54583377.226
producer-metrics:record-queue-time-avg:{client-id=perf-producer-client}                      : 266.010
producer-metrics:request-latency-avg:{client-id=perf-producer-client}                        : 6.109

./kafka-producer-perf-test.sh --topic test_topic --throughput -1 --num-record  23.16s user 4.55s system 97% cpu 28.271 total
grep   0.00s user 0.00s system 0% cpu 28.271 total
```
<br>

## Producer Optimized Performance Test
```
time ./kafka-producer-perf-test.sh \
--topic test_topic \
--throughput -1 \
--num-records 3000000 \
--record-size 1024 \
--producer-props acks=all buffer.memory=67108864 batch.size=100000 linger.ms=100 compression.type=zstd \
bootstrap.servers=kafka-0:19092,kafka-1:29092,kafka-2:39092 --print-metrics | grep \
"3000000 records sent\|\
producer-metrics:outgoing-byte-rate\|\
producer-metrics:bufferpool-wait-ratio\|\
producer-metrics:record-queue-time-avg\|\
producer-metrics:request-latency-avg\|\
producer-metrics:batch-size-avg"

3000000 records sent, 179726.815241 records/sec (175.51 MB/sec), 29.39 ms avg latency, 283.00 ms max latency, 22 ms 50th, 82 ms 95th, 124 ms 99th, 159 ms 99.9th.
producer-metrics:batch-size-avg:{client-id=perf-producer-client}                             : 89082.505
producer-metrics:bufferpool-wait-ratio:{client-id=perf-producer-client}                      : 0.000
producer-metrics:outgoing-byte-rate:{client-id=perf-producer-client}                         : 40844224.775
producer-metrics:record-queue-time-avg:{client-id=perf-producer-client}                      : 16.737
producer-metrics:request-latency-avg:{client-id=perf-producer-client}                        : 9.895

./kafka-producer-perf-test.sh --topic test_topic --throughput -1 --num-record  21.47s user 4.06s system 144% cpu 17.691 total
grep   0.00s user 0.00s system 0% cpu 17.691 total
```
<br>

> Note: Between the above two tests, you can see the total execution time reduce by 2 sec (8.7% improvement). The Avg latency reduced from 272ms to 29.4ms (due to compression and batching).


<br>

## Consumer Performance Test
```
kafka-consumer-perf-test.sh \
--topic test_topic \
--broker-list kafka-0:19092,kafka-1:29092,kafka-2:39092 \
--messages 3000000 | \
jq -R .|jq -sr 'map(./",")|transpose|map(join(": "))[]'

 start.time: 2023-12-30 19:21:02:390
 end.time:  2023-12-30 19:21:32:656
 data.consumed.in.MB:  2930.0850
 MB.sec:  96.8111
 data.consumed.in.nMsg:  3000407
 nMsg.sec:  99134.5734
 rebalance.time.ms:  3174
 fetch.time.ms:  27092
 fetch.MB.sec:  108.1531
 fetch.nMsg.sec:  110748.8188
```
<br>

> Note: kafka-consumer-perf-test.sh doesnt support consumer properties. So you cannot provide properties like ```fetch.min.bytes``` or ```fetch.max.wait.ms```.
<br>
