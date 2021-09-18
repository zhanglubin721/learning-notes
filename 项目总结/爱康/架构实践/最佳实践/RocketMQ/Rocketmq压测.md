# 压测结果

cd /opt/rocketmq/rocketmq-all-4.7.0-bin-release/benchmark



-w 线程数

-s 字符串大小

sh producer.sh -t zms-cluster-perf-test10 -w 10 -s 3072 -n 10.105.160.44:9876
Send TPS: 8413 Max RT: 374 Average RT:  1.170 Send Failed: 0 Response Failed: 0
Send TPS: 9443 Max RT: 374 Average RT:  1.041 Send Failed: 0 Response Failed: 0
Send TPS: 8615 Max RT: 374 Average RT:  1.143 Send Failed: 0 Response Failed: 0
Send TPS: 9458 Max RT: 374 Average RT:  1.041 Send Failed: 0 Response Failed: 0
Send TPS: 8766 Max RT: 374 Average RT:  1.123 Send Failed: 0 Response Failed: 0
Send TPS: 8832 Max RT: 374 Average RT:  1.113 Send Failed: 0 Response Failed: 0
Send TPS: 8832 Max RT: 374 Average RT:  1.113 Send Failed: 0 Response Failed: 0

sh producer.sh -t zms-cluster-perf-test20 -w 20 -s 3072 -n 10.105.160.44:9876
Send TPS: 13823 Max RT: 327 Average RT:  1.429 Send Failed: 0 Response Failed: 0
Send TPS: 14499 Max RT: 327 Average RT:  1.361 Send Failed: 0 Response Failed: 0
Send TPS: 13576 Max RT: 327 Average RT:  1.455 Send Failed: 0 Response Failed: 0
Send TPS: 12907 Max RT: 327 Average RT:  1.530 Send Failed: 0 Response Failed: 0
Send TPS: 12768 Max RT: 327 Average RT:  1.543 Send Failed: 0 Response Failed: 0
Send TPS: 13314 Max RT: 327 Average RT:  1.484 Send Failed: 0 Response Failed: 0
Send TPS: 13174 Max RT: 327 Average RT:  1.500 Send Failed: 0 Response Failed: 0

sh producer.sh -t zms-cluster-perf-test30 -w 30 -s 3072 -n 10.105.160.44:9876
Send TPS: 15038 Max RT: 321 Average RT:  1.977 Send Failed: 0 Response Failed: 0
Send TPS: 14785 Max RT: 321 Average RT:  2.011 Send Failed: 0 Response Failed: 0
Send TPS: 14458 Max RT: 321 Average RT:  2.057 Send Failed: 0 Response Failed: 0
Send TPS: 14518 Max RT: 321 Average RT:  2.048 Send Failed: 0 Response Failed: 0
Send TPS: 14574 Max RT: 321 Average RT:  2.043 Send Failed: 0 Response Failed: 0
Send TPS: 14710 Max RT: 321 Average RT:  2.022 Send Failed: 0 Response Failed: 0
Send TPS: 15051 Max RT: 321 Average RT:  1.976 Send Failed: 0 Response Failed: 0

sh producer.sh -t zms-cluster-perf-test50 -w 50 -s 3072 -n 10.105.160.44:9876
Send TPS: 16107 Max RT: 358 Average RT:  3.083 Send Failed: 0 Response Failed: 0
Send TPS: 17046 Max RT: 358 Average RT:  2.916 Send Failed: 0 Response Failed: 0
Send TPS: 16918 Max RT: 358 Average RT:  2.935 Send Failed: 0 Response Failed: 0
Send TPS: 16865 Max RT: 358 Average RT:  2.946 Send Failed: 0 Response Failed: 0
Send TPS: 17288 Max RT: 358 Average RT:  2.869 Send Failed: 0 Response Failed: 0
Send TPS: 17321 Max RT: 358 Average RT:  2.866 Send Failed: 0 Response Failed: 0
Send TPS: 16259 Max RT: 358 Average RT:  3.055 Send Failed: 0 Response Failed: 0

sh producer.sh -t zms-cluster-perf-test100 -w 100 -s 3072 -n 10.105.160.44:9876
Send TPS: 18435 Max RT: 494 Average RT:  5.402 Send Failed: 0 Response Failed: 0
Send TPS: 19336 Max RT: 494 Average RT:  5.151 Send Failed: 0 Response Failed: 0
Send TPS: 18268 Max RT: 494 Average RT:  5.454 Send Failed: 0 Response Failed: 0
Send TPS: 18645 Max RT: 494 Average RT:  5.336 Send Failed: 0 Response Failed: 0
Send TPS: 14156 Max RT: 494 Average RT:  7.038 Send Failed: 0 Response Failed: 0
Send TPS: 18653 Max RT: 494 Average RT:  5.336 Send Failed: 0 Response Failed: 0
Send TPS: 19093 Max RT: 494 Average RT:  5.213 Send Failed: 0 Response Failed: 0
Send TPS: 19150 Max RT: 494 Average RT:  5.198 Send Failed: 0 Response Failed: 0