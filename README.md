Link do wpisu: https://wiadrodanych.pl/big-data/deduplikacja-zdarzen-logstash-redis/

![](https://github.com/zorteran/logstash-deduplication/blob/d603fd67dbe7cc9922b0c6d176fd3430e899a970/demo.gif)

本来想借鉴此方法，后面发现用logstash内置filter：fingerprint + memcached，可以实现去重功能

```

  ### fingerprint 和 memcached 实现去重(dedupe)功能begin
  fingerprint {
    source => [ "cjdate", "gddm", "zqdm", "wtid", "cjid" ]
    target => "[fingerprint]"
    concatenate_sources => true
    method => "SHA1"
  }

  memcached {
      id => "memcached-getfp"
      hosts => ["127.0.0.1:11211"]
      namespace => "dedupe"
      get => {"%{fingerprint}" => "[cjexists]"}
  }

  if [cjexists] {
    mutate { add_tag => [ "dedupe" ] }      # memcached 根据 fingerprint 如果查回来cjexists字段，则表面为重复记录，加tag，也可以直接drop{}
  } else {
    memcached {
      hosts => ["127.0.0.1:11211"]
      namespace => "dedupe"
      ttl => 28800                          # ttl一定要配否则内存爆，28800秒=8小时
      set => { "gddm" => "%{fingerprint}" } # 存什么字段不重要，这里是存的股东代码
      id => "memcached-setfp"
   }
  }
  ### fingerprint 和 memcached 实现去重(dedupe)功能end
  
```
