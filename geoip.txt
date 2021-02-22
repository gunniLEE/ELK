[root@elk-master ELK_LABs]# vi weblog-skcnc.log

14.49.42.25 - - [01/Feb/2021:00:04:05 +0900] "GET /tag/661 HTTP/1.1" 200 39781 "-" "Mozilla/5.0 (compatible; AhrefsBot/4.0; +http://ahrefs.com/robot/)"
173.199.120.19 - - [01/Feb/2021:00:04:05 +0900] "GET /tag/661 HTTP/1.1" 200 39781 "-" "Mozilla/5.0 (compatible; AhrefsBot/4.0; +http://ahrefs.com/robot/)"
216.244.66.246 - - [30/Apr/2020:04:28:11 +0000] "GET /docs/triton/pages.html HTTP/1.1" 200 5639 "-" "Mozilla/5.0 (compatible; DotBot/1.1; http://www.opensiteexplorer.org/dotbot, help@moz.com)"
199.21.99.207 - - [30/Apr/2020:04:29:44 +0000] "GET /docs/triton/class_triton_1_1_wind_fetch-members.html HTTP/1.1" 200 1845 "-" "Mozilla/5.0 (compatible; YandexBot/3.0; +http://yandex.com/bots)"
199.21.99.207 - - [30/Apr/2020:04:29:57 +0000] "GET /docs/triton/class_triton_1_1_breaking_waves_parameters-members.html HTTP/1.1" 200 1967 "-" "Mozilla/5.0 (compatible; YandexBot/3.0; +http://yandex.com/bots)"
100.43.90.9 - - [30/Apr/2020:04:30:13 +0000] "GET /docs/html/functions_rela.html HTTP/1.1" 200 882 "-" "Mozilla/5.0 (compatible; YandexBot/3.0; +http://yandex.com/bots)"
217.182.132.36 - - [30/Apr/2020:04:30:31 +0000] "GET /2012/08/sundog-software-featured-in-august-2012-issue-of-develop/ HTTP/1.1" 200 14326 "-" "Mozilla/5.0 (compatible; AhrefsBot/5.2; +http://ahrefs.com/robot/)"
54.210.20.202 - - [30/Apr/2020:04:30:58 +0000] "POST /wp-cron.php?doing_wp_cron=1493526658.6017329692840576171875 HTTP/1.1" 200 - "http://sundog-soft.com/wp-cron.php?doing_wp_cron=1493526658.6017329692840576171875" "WordPress/4.7.4; http://sundog-soft.com"
77.247.181.163 - - [30/Apr/2020:04:30:57 +0000] "GET / HTTP/1.1" 301 - "-" "Mozilla/5.0 (Windows NT 5.1; rv:7.0.1) Gecko/20100101 Firefox/7.0.1"
"weblog-skcnc.log" 13L, 2269C

[root@elk-master ~]# cd /etc/logstash/conf.d/

[root@elk-master conf.d]# vi weblog-skcnc.conf

input {
  tcp {
    port => 8899
    type => "apache"
  }
}

filter {
  if [type] == "apache" {

    grok {
      match => {
        message => "%{COMBINEDAPACHELOG}"
        remove_field => "message"
      }
    }

    geoip {
      source => "clientip"
      fields => [ "city_name", "country_name", "location", "region_name"]
    }

    date {
      match => ["timestamp", "dd/MMM/yyyy:HH:mm:ss Z"]
      remove_field => "timestamp"
    }
  }
}

output {
  elasticsearch {
    hosts => ["localhost:9200"]
  }
}

[root@elk-master conf.d]# /usr/share/logstash/bin/logstash -f weblog-skcnc.conf

[root@elk-master ~]# curl localhost:9200/_cat/indices?v | grep logstash

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  3302  100  3302    0     0   8854      0 --:--:-- --:--:-- --:--:--  8876
yellow open   logstash-2021.02.22-000001      WeQbvvlgTTKngHEcJvYiHw   1   1                                                                                                                       0            0       208b           208b

[root@elk-master ~]# cd ELK_LABs/
[root@elk-master ELK_LABs]# cat weblog-skcnc.log | nc localhost 8899
