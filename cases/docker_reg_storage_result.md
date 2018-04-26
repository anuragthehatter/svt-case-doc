# Results from Docker Registry Storage Tests

# Push time

```sh
# grep "Failed builds: " /tmp/build_test.log -A5          
2017-12-04 18:28:34,724 - build_test - MainThread - INFO - Failed builds: 0
2017-12-04 18:28:34,724 - build_test - MainThread - INFO - Invalid builds: 1
2017-12-04 18:28:34,724 - build_test - MainThread - INFO - Good builds included in stats: 499
2017-12-04 18:28:34,724 - build_test - MainThread - INFO - Average build time, all good builds: 122
2017-12-04 18:28:34,724 - build_test - MainThread - INFO - Minimum build time, all good builds: 48
2017-12-04 18:28:34,724 - build_test - MainThread - INFO - Maximum build time, all good builds: 165
--
2017-12-04 18:44:02,373 - build_test - MainThread - INFO - Failed builds: 1
2017-12-04 18:44:02,373 - build_test - MainThread - INFO - Invalid builds: 0
2017-12-04 18:44:02,374 - build_test - MainThread - INFO - Good builds included in stats: 499
2017-12-04 18:44:02,374 - build_test - MainThread - INFO - Average build time, all good builds: 118
2017-12-04 18:44:02,374 - build_test - MainThread - INFO - Minimum build time, all good builds: 47
2017-12-04 18:44:02,374 - build_test - MainThread - INFO - Maximum build time, all good builds: 164
--
2017-12-04 19:06:09,226 - build_test - MainThread - INFO - Failed builds: 0
2017-12-04 19:06:09,226 - build_test - MainThread - INFO - Invalid builds: 0
2017-12-04 19:06:09,226 - build_test - MainThread - INFO - Good builds included in stats: 500
2017-12-04 19:06:09,226 - build_test - MainThread - INFO - Average build time, all good builds: 114
2017-12-04 19:06:09,226 - build_test - MainThread - INFO - Minimum build time, all good builds: 44
2017-12-04 19:06:09,226 - build_test - MainThread - INFO - Maximum build time, all good builds: 165
--
2017-12-04 19:51:10,432 - build_test - MainThread - INFO - Failed builds: 1
2017-12-04 19:51:10,432 - build_test - MainThread - INFO - Invalid builds: 0
2017-12-04 19:51:10,432 - build_test - MainThread - INFO - Good builds included in stats: 999
2017-12-04 19:51:10,433 - build_test - MainThread - INFO - Average build time, all good builds: 205
2017-12-04 19:51:10,433 - build_test - MainThread - INFO - Minimum build time, all good builds: 44
2017-12-04 19:51:10,433 - build_test - MainThread - INFO - Maximum build time, all good builds: 380
--
2017-12-04 20:49:06,765 - build_test - MainThread - INFO - Failed builds: 86
2017-12-04 20:49:06,766 - build_test - MainThread - INFO - Invalid builds: 15
2017-12-04 20:49:06,766 - build_test - MainThread - INFO - Good builds included in stats: 885
2017-12-04 20:49:06,766 - build_test - MainThread - INFO - Average build time, all good builds: 361
2017-12-04 20:49:06,766 - build_test - MainThread - INFO - Minimum build time, all good builds: 46
2017-12-04 20:49:06,766 - build_test - MainThread - INFO - Maximum build time, all good builds: 1123

# grep "Failed builds: " /tmp/build_test.log -A5 
2017-12-05 20:17:48,397 - build_test - MainThread - INFO - Failed builds: 1
2017-12-05 20:17:48,397 - build_test - MainThread - INFO - Invalid builds: 0
2017-12-05 20:17:48,397 - build_test - MainThread - INFO - Good builds included in stats: 499
2017-12-05 20:17:48,397 - build_test - MainThread - INFO - Average build time, all good builds: 128
2017-12-05 20:17:48,397 - build_test - MainThread - INFO - Minimum build time, all good builds: 53
2017-12-05 20:17:48,397 - build_test - MainThread - INFO - Maximum build time, all good builds: 167
--
2017-12-05 20:32:21,834 - build_test - MainThread - INFO - Failed builds: 0
2017-12-05 20:32:21,834 - build_test - MainThread - INFO - Invalid builds: 0
2017-12-05 20:32:21,834 - build_test - MainThread - INFO - Good builds included in stats: 500
2017-12-05 20:32:21,834 - build_test - MainThread - INFO - Average build time, all good builds: 122
2017-12-05 20:32:21,835 - build_test - MainThread - INFO - Minimum build time, all good builds: 47
2017-12-05 20:32:21,835 - build_test - MainThread - INFO - Maximum build time, all good builds: 160
--
2017-12-05 20:41:25,096 - build_test - MainThread - INFO - Failed builds: 1
2017-12-05 20:41:25,096 - build_test - MainThread - INFO - Invalid builds: 0
2017-12-05 20:41:25,096 - build_test - MainThread - INFO - Good builds included in stats: 499
2017-12-05 20:41:25,096 - build_test - MainThread - INFO - Average build time, all good builds: 116
2017-12-05 20:41:25,096 - build_test - MainThread - INFO - Minimum build time, all good builds: 42
2017-12-05 20:41:25,096 - build_test - MainThread - INFO - Maximum build time, all good builds: 176
--
2017-12-05 21:59:55,970 - build_test - MainThread - INFO - Failed builds: 1
2017-12-05 21:59:55,971 - build_test - MainThread - INFO - Invalid builds: 0
2017-12-05 21:59:55,971 - build_test - MainThread - INFO - Good builds included in stats: 999
2017-12-05 21:59:55,971 - build_test - MainThread - INFO - Average build time, all good builds: 204
2017-12-05 21:59:55,971 - build_test - MainThread - INFO - Minimum build time, all good builds: 48
2017-12-05 21:59:55,971 - build_test - MainThread - INFO - Maximum build time, all good builds: 379
--
2017-12-06 01:50:00,195 - build_test - MainThread - INFO - Failed builds: 0
2017-12-06 01:50:00,195 - build_test - MainThread - INFO - Invalid builds: 0
2017-12-06 01:50:00,195 - build_test - MainThread - INFO - Good builds included in stats: 1000
2017-12-06 01:50:00,195 - build_test - MainThread - INFO - Average build time, all good builds: 269
2017-12-06 01:50:00,196 - build_test - MainThread - INFO - Minimum build time, all good builds: 41
2017-12-06 01:50:00,196 - build_test - MainThread - INFO - Maximum build time, all good builds: 442

[fedora@ip-172-31-55-221 ~]$ grep "Failed builds: " /tmp/build_test.log -A5  
2018-04-25 20:56:52,213 - build_test - MainThread - INFO - Failed builds: 0
2018-04-25 20:56:52,213 - build_test - MainThread - INFO - Invalid builds: 0
2018-04-25 20:56:52,213 - build_test - MainThread - INFO - Good builds included in stats: 500
2018-04-25 20:56:52,213 - build_test - MainThread - INFO - Average build time, all good builds: 138
2018-04-25 20:56:52,213 - build_test - MainThread - INFO - Minimum build time, all good builds: 62
2018-04-25 20:56:52,213 - build_test - MainThread - INFO - Maximum build time, all good builds: 177
--
2018-04-25 21:47:28,293 - build_test - MainThread - INFO - Failed builds: 0
2018-04-25 21:47:28,294 - build_test - MainThread - INFO - Invalid builds: 0
2018-04-25 21:47:28,294 - build_test - MainThread - INFO - Good builds included in stats: 500
2018-04-25 21:47:28,294 - build_test - MainThread - INFO - Average build time, all good builds: 141
2018-04-25 21:47:28,294 - build_test - MainThread - INFO - Minimum build time, all good builds: 60
2018-04-25 21:47:28,294 - build_test - MainThread - INFO - Maximum build time, all good builds: 191
--
2018-04-25 22:56:26,000 - build_test - MainThread - INFO - Failed builds: 0
2018-04-25 22:56:26,000 - build_test - MainThread - INFO - Invalid builds: 0
2018-04-25 22:56:26,000 - build_test - MainThread - INFO - Good builds included in stats: 500
2018-04-25 22:56:26,000 - build_test - MainThread - INFO - Average build time, all good builds: 148
2018-04-25 22:56:26,000 - build_test - MainThread - INFO - Minimum build time, all good builds: 71
2018-04-25 22:56:26,001 - build_test - MainThread - INFO - Maximum build time, all good builds: 198
--
2018-04-26 01:39:29,175 - build_test - MainThread - INFO - Failed builds: 0
2018-04-26 01:39:29,175 - build_test - MainThread - INFO - Invalid builds: 0
2018-04-26 01:39:29,175 - build_test - MainThread - INFO - Good builds included in stats: 1000
2018-04-26 01:39:29,176 - build_test - MainThread - INFO - Average build time, all good builds: 333
2018-04-26 01:39:29,176 - build_test - MainThread - INFO - Minimum build time, all good builds: 98
2018-04-26 01:39:29,176 - build_test - MainThread - INFO - Maximum build time, all good builds: 434
--
2018-04-26 02:12:35,563 - build_test - MainThread - INFO - Failed builds: 0
2018-04-26 02:12:35,563 - build_test - MainThread - INFO - Invalid builds: 0
2018-04-26 02:12:35,563 - build_test - MainThread - INFO - Good builds included in stats: 1000
2018-04-26 02:12:35,563 - build_test - MainThread - INFO - Average build time, all good builds: 359
2018-04-26 02:12:35,563 - build_test - MainThread - INFO - Minimum build time, all good builds: 123
2018-04-26 02:12:35,563 - build_test - MainThread - INFO - Maximum build time, all good builds: 479
--
2018-04-26 02:50:25,950 - build_test - MainThread - INFO - Failed builds: 0
2018-04-26 02:50:25,950 - build_test - MainThread - INFO - Invalid builds: 0
2018-04-26 02:50:25,950 - build_test - MainThread - INFO - Good builds included in stats: 1000
2018-04-26 02:50:25,951 - build_test - MainThread - INFO - Average build time, all good builds: 381
2018-04-26 02:50:25,951 - build_test - MainThread - INFO - Minimum build time, all good builds: 126
2018-04-26 02:50:25,951 - build_test - MainThread - INFO - Maximum build time, all good builds: 496


```
