# fluent-plugin-mackerel [![Build Status](https://travis-ci.org/tksmd/fluent-plugin-mackerel.png?branch=master)](https://travis-ci.org/tksmd/fluent-plugin-mackerel)

## Overview

[Fluentd](http://fluentd.org) plugin to send metrics to [mackerel.io](http://mackerel.io/).

This plugin includes two components, namely MackerelOutput and MackerelHostidTagOutput. The former is used to send metrics to mackerel and the latter is used to append mackerel hostid to tag or record.

## Installation

Install with gem or fluent-gem command as:

```
# for fluentd
$ gem install fluent-plugin-mackerel

# for td-agent
$ sudo /usr/lib64/fluent/ruby/bin/fluent-gem install fluent-plugin-mackerel
```

## Configuration

### MackerelOutput

This plugin uses [APIv0](http://help-ja.mackerel.io/entry/spec/api/v0) of mackerel.io.
```
<match ...>
  type mackerel
  api_key 123456
  hostid xyz
  metrics_name http_status.${out_key}
  out_keys 2xx_count,3xx_count,4xx_count,5xx_count
</match>
```

Then the sent metric data will look like this:
```
{
  "hostId": "xyz",
  "name": "custom.http_status.2xx_count",
  "time": 1399997498,
  "value": 100.0
}
```
As shown above, `${out_key}` will be replaced with out_key like "2xx_count" when sending metrics.

You can use `out_key_pattern` instead of `out_keys`. Input records whose key matches the pattern set to `out_key_pattern` will be sent. Either `out_keys` or `out_key_pattern` is required.

```
<match ...>
  type mackerel
  api_key 123456
  service yourservice
  metrics_name http_status.${out_key}
  out_key_pattern [2-5]xx_count
```

You can use `${[n]}` for `metrics_name` where `n` represents any decimal number including negative value,

```
<match mackerel.*>
  type mackerel
  api_key 123456
  hostid xyz
  metrics_name ${[1]}.${out_key}
  out_keys 2xx_count,3xx_count,4xx_count,5xx_count
</match>
```

then it indicates value of the `n` th index of the array got by splitting the tag by `.` (dot) and get the following output when the tag is `mackerel.http_status`.
```
{
  "hostId": "xyz",
  "name": "custom.http_status.2xx_count",
  "time": 1399997498,
  "value": 100.0
}
```
"custom" will be appended to the `name` attribute before sending metrics to mackerel automatically.

You can also send ["service" metric](http://help-ja.mackerel.io/entry/spec/api/v0#service-metric-value-post) as follows.
```
<match ...>
  type mackerel
  api_key 123456
  service yourservice
  metrics_name http_status.${out_key}
  out_keys 2xx_count,3xx_count,4xx_count,5xx_count
</match>
```

When you send service metric, "custom" can be removed with `remove_prefix` as follows.
This option is not availabe when sending host metric.

```
<match ...>
  type mackerel
  api_key 123456
  service yourservice
  remove_prefix
  metrics_name http_status.${out_key}
  out_keys 2xx_count,3xx_count,4xx_count,5xx_count
</match>
```

`flush_interval` is not allowed to be set less than 60 secs not to send API requests more than once in a minute.

This plugin overwrites the default value of `buffer_queue_limit` and `buffer_chunk_limit` as follows.

* buffer_queue_limit to 4096
* buffer_chunk_limit to 100K

Without any special reasons to change, you should leave them as they are.

Since version 0.0.4, metrics_prefix was removed and you should use metrics_name instead.

### MackerelHostidTagOutput

If you want to add the hostid to the record with a certain key name, do the following.
```
<match ...>
  type mackerel_hostid_tag
  add_to record
  key_name mackerel_hostid
</match>
```
As shown above, key_name field is required. Supposed host_id is "xyz" and input is `["test", 1407650400, {"val1"=>1, "val2"=>2, "val3"=>3, "val4"=>4}]`, then you can get `["test", 1407650400, {"val1"=>1, "val2"=>2, "val3"=>3, "val4"=>4, "mackerel_hostid"=>"xyz"}]`

To append hostid to the tag, you can simply configure "add_to" as "tag" like this.
```
<match ...>
  type mackerel_hostid_tag
  add_to tag
</match>
```
When input is `["test", 1407650400, {"val1"=>1, "val2"=>2, "val3"=>3, "val4"=>4}]`, then the output will be `["test.xyz", 1407650400, {"val1"=>1, "val2"=>2, "val3"=>3, "val4"=>4}]`

## TODO

Pull requests are very welcome!!

## For developers

You have to run the command below when starting development.
```
$ bundle install --path vendor/bundle
```

To run tests, do the following.
```
$ VERBOSE=1 bundle exec rake test
```

If you want to run a certain file, run rake like this
```
$ VERBOSE=1 bundle exec rake test TEST=test/plugin/test_out_mackerel.rb
```

In addition, you can run specific method like this.
```
$ VERBOSE=1 bundle exec rake test TEST=test/plugin/test_out_mackerel.rb TESTOPTS="--name=test_configure"
```

When releasing, call rake release as follows.
```
$ bundle exec rake release
```

For debugging purpose, you can change Mackerel endpoint by `origin` parameter like this.
```
<match ...>
  type mackerel
  api_key 123456
  hostid xyz
  metrics_name http_status.${out_key}
  out_keys 2xx_count,3xx_count,4xx_count,5xx_count
  origin https://example.com
</match>
```

## References

* [Posting Service Metrics with fluentd](http://help.mackerel.io/entry/advanced/fluentd)
* [How to use fluent-plugin-mackerel (Japanese)](http://qiita.com/tksmd/items/1212331a5a18afe520df)

## Authors

- Takashi Someda ([@tksmd](http://twitter.com/tksmd/))
- Mackerel Development Team

## Copyright

* Copyright (c) 2014- Hatena Co., Ltd. and Authors
* Apache License, Version 2.0
