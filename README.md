# String allocation in active record with PG adapter

I've notice arround 112 empty string allocation on my app when pg adapter when doing one controller call over one model with few records.

With 100 calls we have :
```
Allocated String Report
-----------------------
      1500  ""
       400  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/connection_adapters/postgresql_adapter.rb:427
        ..  ...
```

When freezing `''` no more too much allocation (test with 100 requests) :

```
      1100  ""
       300  /Users/bti/.rvm/rubies/ruby-2.4.0/lib/ruby/2.4.0/logger.rb:598
       200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_dispatch/http/request.rb:228
       100  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_dispatch/http/request.rb:141
       100  /Users/bti/.rvm/gems/ruby-2.4.0/gems/rack-2.0.1/lib/rack/mock.rb:102
       100  /Users/bti/.rvm/gems/ruby-2.4.0/gems/rack-2.0.1/lib/rack/mock.rb:107
       100  /Users/bti/.rvm/gems/ruby-2.4.0/gems/rack-2.0.1/lib/rack/mock.rb:131
       100  /Users/bti/.rvm/gems/ruby-2.4.0/gems/rack-2.0.1/lib/rack/mock.rb:97
       100  /Users/bti/.rvm/gems/ruby-2.4.0/gems/railties-5.0.1/lib/rails/rack/logger.rb:52
```

------

I reproduced the step using Rails 5 and activerecord-5.0.1.

Before :
```
$ RAILS_ENV=production rc
2.4.0 :001 > User.count
 => 0
2.4.0 :002 > 100.times{ User.create(name: Faker::Name.name) }
 => 100
2.4.0 :003 > User.count
 => 100
```

Then :
```sh
$ PATH_TO_HIT='http://localhost:3000/users.json' bundle exec derailed exec perf:objects
```

## Result with only 1 invocation

Result :
```
Booting: production
Endpoint: "http://localhost:3000/users.json"
Running 1 times
Total allocated: 1080938 bytes (17900 objects)
Total retained:  0 bytes (0 objects)

allocated memory by gem
-----------------------------------
    460360  activesupport-5.0.1
    245680  activerecord-5.0.1
    242976  activemodel-5.0.1
     78709  ruby-2.4.0/lib
     28800  tzinfo-1.2.2
     13536  actionpack-5.0.1
      7410  rack-2.0.1
      1491  railties-5.0.1
       912  arel-7.1.4
       560  actionview-5.0.1
       272  string_allocation/app
       232  concurrent-ruby-1.0.4

allocated memory by file
-----------------------------------
    184576  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activemodel-5.0.1/lib/active_model/type/helpers/time_value.rb
    159464  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb
    115200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/time_with_zone.rb
     84040  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb
     84000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/attribute_set/builder.rb
     74938  /Users/bti/.rvm/rubies/ruby-2.4.0/lib/ruby/2.4.0/json/common.rb
     54400  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activemodel-5.0.1/lib/active_model/serialization.rb
     32000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/attribute.rb
     32000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/acts_like.rb
     27200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/try.rb
     24000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/serialization.rb
     20640  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/result.rb
     19704  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/connection_adapters/postgresql/database_statements.rb
     19200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/persistence.rb
     19200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/hash/transform_values.rb
     16880  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/core.rb
     16000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/attribute_methods/time_zone_conversion.rb
     14400  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/date_and_time/zones.rb
     14400  /Users/bti/.rvm/gems/ruby-2.4.0/gems/tzinfo-1.2.2/lib/tzinfo/time_or_datetime.rb
     14400  /Users/bti/.rvm/gems/ruby-2.4.0/gems/tzinfo-1.2.2/lib/tzinfo/transition_data_timezone_info.rb
      4000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activemodel-5.0.1/lib/active_model/type/string.rb
      4000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/aggregations.rb
      4000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/associations.rb
      3080  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_dispatch/http/response.rb
      2968  /Users/bti/.rvm/gems/ruby-2.4.0/gems/rack-2.0.1/lib/rack/mock.rb
      2696  /Users/bti/.rvm/gems/ruby-2.4.0/gems/rack-2.0.1/lib/rack/utils.rb
      1974  /Users/bti/.rvm/rubies/ruby-2.4.0/lib/ruby/2.4.0/logger.rb
      1808  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/tagged_logging.rb
      1704  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/querying.rb
      1560  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_dispatch/http/parameter_filter.rb
      1347  /Users/bti/.rvm/gems/ruby-2.4.0/gems/railties-5.0.1/lib/rails/rack/logger.rb
      1344  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/callbacks.rb
      1224  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/subscriber.rb
      1208  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_controller/metal/instrumentation.rb
      1176  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/notifications/instrumenter.rb
      1040  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/connection_adapters/postgresql_adapter.rb
       904  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/hash_with_indifferent_access.rb
       880  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_dispatch/journey/gtg/transition_table.rb
       864  /Users/bti/.rvm/rubies/ruby-2.4.0/lib/ruby/2.4.0/uri/rfc2396_parser.rb
       840  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_controller/log_subscriber.rb
       826  /Users/bti/.rvm/gems/ruby-2.4.0/gems/rack-2.0.1/lib/rack/etag.rb
       680  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_dispatch/http/mime_negotiation.rb
       664  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_dispatch/journey/router.rb
       592  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_dispatch/journey/gtg/simulator.rb
       557  /Users/bti/.rvm/rubies/ruby-2.4.0/lib/ruby/2.4.0/securerandom.rb
       528  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/connection_adapters/abstract_adapter.rb
       520  /Users/bti/.rvm/gems/ruby-2.4.0/gems/rack-2.0.1/lib/rack/request.rb
       480  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/inflector/methods.rb
       440  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_dispatch/http/request.rb
       440  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_dispatch/journey/path/pattern.rb

allocated memory by location
-----------------------------------
     99200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/time_with_zone.rb:481
     74714  /Users/bti/.rvm/rubies/ruby-2.4.0/lib/ruby/2.4.0/json/common.rb:224
     71200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:89
     69376  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activemodel-5.0.1/lib/active_model/type/helpers/time_value.rb:69
     64000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34
     48000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activemodel-5.0.1/lib/active_model/type/helpers/time_value.rb:70
     48000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activemodel-5.0.1/lib/active_model/type/helpers/time_value.rb:71
     45600  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:85
     41400  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
     32800  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/attribute_set/builder.rb:64
     32000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/attribute.rb:5
     32000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/acts_like.rb:8
     24000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activemodel-5.0.1/lib/active_model/serialization.rb:131
     20040  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:142
     19312  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/connection_adapters/postgresql/database_statements.rb:111
     19200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activemodel-5.0.1/lib/active_model/serialization.rb:134
     19200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activemodel-5.0.1/lib/active_model/type/helpers/time_value.rb:61
     19200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/attribute_set/builder.rb:33
     19200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/attribute_set/builder.rb:65
     19200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/persistence.rb:69
     19200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/result.rb:123
     19200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/hash/transform_values.rb:14
     19200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/try.rb:17
     16000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/attribute_methods/time_zone_conversion.rb:8
     16000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/time_with_zone.rb:149
     14400  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/date_and_time/zones.rb:33
     14400  /Users/bti/.rvm/gems/ruby-2.4.0/gems/tzinfo-1.2.2/lib/tzinfo/time_or_datetime.rb:314
     14400  /Users/bti/.rvm/gems/ruby-2.4.0/gems/tzinfo-1.2.2/lib/tzinfo/transition_data_timezone_info.rb:115
     12800  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/core.rb:135
      8800  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/attribute_set/builder.rb:19
      8000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/serialization.rb:12
      8000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/serialization.rb:14
      8000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/serialization.rb:15
      8000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/try.rb:6
      7200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activemodel-5.0.1/lib/active_model/serialization.rb:127
      4000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activemodel-5.0.1/lib/active_model/serialization.rb:137
      4000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activemodel-5.0.1/lib/active_model/type/string.rb:15
      4000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/aggregations.rb:23
      4000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/associations.rb:268
      4000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/attribute_set/builder.rb:20
      4000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/core.rb:546
      1760  /Users/bti/.rvm/gems/ruby-2.4.0/gems/rack-2.0.1/lib/rack/mock.rb:92
      1064  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/querying.rb:50
      1056  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/notifications/instrumenter.rb:58
       968  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_dispatch/http/response.rb:412
       888  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/tagged_logging.rb:54
       848  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_dispatch/http/parameter_filter.rb:40
       843  /Users/bti/.rvm/rubies/ruby-2.4.0/lib/ruby/2.4.0/logger.rb:695
       840  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/result.rb:118
       840  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:91

allocated memory by class
-----------------------------------
    266656  Array
    243442  String
    194912  Hash
    104000  Rational
     64000  ActiveSupport::JSON::Encoding::JSONGemEncoder::EscapedString
     57680  MatchData
     53760  Time
     32000  ActiveRecord::Attribute::FromDatabase
     14400  TZInfo::TimeOrDateTime
     14400  TZInfo::TimezonePeriod
     12800  User
      8800  ActiveRecord::LazyAttributeHash
      4000  ActiveRecord::AttributeSet
      2320  Proc
      1016  ActionDispatch::Request
      1008  Rack::Utils::HeaderHash
       576  ActiveSupport::HashWithIndifferentAccess
       512  Regexp
       456  Class
       360  ActiveSupport::Callbacks::Filters::Environment
       288  ActiveSupport::Notifications::Event
       272  StringScanner
       232  <<Unknown>>
       224  JSON::Ext::Generator::State
       216  ActionDispatch::Response::Buffer
       176  Thread::Mutex
       160  Rack::BodyProxy
       160  StringIO
       144  ActiveRecord::Result
       136  ActionDispatch::Response
       136  UsersController
       120  ActionDispatch::Response::ContentTypeHeader
       120  URI::HTTP
       112  ActiveRecord::Relation
       104  Arel::Nodes::SelectCore
       104  Rack::MockResponse
        88  ActionView::LookupContext
        88  Arel::Nodes::SelectStatement
        80  Digest::SHA256
        80  PG::Result
        40  ActionDispatch::Http::Headers
        40  ActionDispatch::Http::ParameterFilter
        40  ActionDispatch::Http::ParameterFilter::CompiledFilter
        40  ActionDispatch::Journey::Path::Pattern::MatchData
        40  ActionDispatch::RemoteIp::GetIp
        40  ActionDispatch::Response::Header
        40  ActionDispatch::Response::RackBody
        40  ActionView::PathSet
        40  ActiveRecord::Associations::Preloader
        40  ActiveRecord::ConnectionAdapters::AbstractAdapter::SQLString

allocated objects by gem
-----------------------------------
      8613  activesupport-5.0.1
      3600  activemodel-5.0.1
      3393  activerecord-5.0.1
      1548  ruby-2.4.0/lib
       400  tzinfo-1.2.2
       209  actionpack-5.0.1
        85  rack-2.0.1
        20  arel-7.1.4
        18  railties-5.0.1
         9  actionview-5.0.1
         3  string_allocation/app
         2  concurrent-ruby-1.0.4

allocated objects by file
-----------------------------------
      3000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activemodel-5.0.1/lib/active_model/type/helpers/time_value.rb
      2804  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb
      2600  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/time_with_zone.rb
      1701  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb
      1502  /Users/bti/.rvm/rubies/ruby-2.4.0/lib/ruby/2.4.0/json/common.rb
       900  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/attribute_set/builder.rb
       800  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/acts_like.rb
       600  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/serialization.rb
       500  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activemodel-5.0.1/lib/active_model/serialization.rb
       408  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/connection_adapters/postgresql/database_statements.rb
       400  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/attribute.rb
       400  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/attribute_methods/time_zone_conversion.rb
       300  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/try.rb
       202  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/core.rb
       200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/date_and_time/zones.rb
       200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/tzinfo-1.2.2/lib/tzinfo/time_or_datetime.rb
       200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/tzinfo-1.2.2/lib/tzinfo/transition_data_timezone_info.rb
       109  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/result.rb
       100  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activemodel-5.0.1/lib/active_model/type/string.rb
       100  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/aggregations.rb
       100  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/associations.rb
       100  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/persistence.rb
       100  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/hash/transform_values.rb
        49  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_dispatch/http/response.rb
        28  /Users/bti/.rvm/gems/ruby-2.4.0/gems/rack-2.0.1/lib/rack/utils.rb
        22  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/connection_adapters/postgresql_adapter.rb
        21  /Users/bti/.rvm/rubies/ruby-2.4.0/lib/ruby/2.4.0/logger.rb
        20  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/tagged_logging.rb
        20  /Users/bti/.rvm/gems/ruby-2.4.0/gems/rack-2.0.1/lib/rack/mock.rb
        18  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/subscriber.rb
        17  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_dispatch/http/parameter_filter.rb
        17  /Users/bti/.rvm/gems/ruby-2.4.0/gems/railties-5.0.1/lib/rails/rack/logger.rb
        16  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_dispatch/journey/gtg/transition_table.rb
        16  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/callbacks.rb
        15  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/hash_with_indifferent_access.rb
        15  /Users/bti/.rvm/gems/ruby-2.4.0/gems/rack-2.0.1/lib/rack/etag.rb
        13  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_controller/metal/instrumentation.rb
        13  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_dispatch/http/mime_negotiation.rb
        13  /Users/bti/.rvm/gems/ruby-2.4.0/gems/rack-2.0.1/lib/rack/request.rb
        12  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_controller/log_subscriber.rb
        10  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/numeric/conversions.rb
        10  /Users/bti/.rvm/rubies/ruby-2.4.0/lib/ruby/2.4.0/uri/rfc2396_parser.rb
         9  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_dispatch/http/request.rb
         9  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_dispatch/journey/gtg/simulator.rb
         9  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_dispatch/journey/router.rb
         9  /Users/bti/.rvm/rubies/ruby-2.4.0/lib/ruby/2.4.0/securerandom.rb
         8  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_controller/metal/rendering.rb
         7  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/connection_adapters/abstract_adapter.rb
         7  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/querying.rb
         7  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/relation/query_methods.rb

allocated objects by location
-----------------------------------
      2200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/time_with_zone.rb:481
      1600  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34
      1501  /Users/bti/.rvm/rubies/ruby-2.4.0/lib/ruby/2.4.0/json/common.rb:224
      1200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activemodel-5.0.1/lib/active_model/type/helpers/time_value.rb:70
      1200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activemodel-5.0.1/lib/active_model/type/helpers/time_value.rb:71
      1000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:89
       900  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
       900  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:85
       800  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/acts_like.rb:8
       500  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/attribute_set/builder.rb:64
       402  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/connection_adapters/postgresql/database_statements.rb:111
       400  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activemodel-5.0.1/lib/active_model/type/helpers/time_value.rb:69
       400  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/attribute.rb:5
       400  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/attribute_methods/time_zone_conversion.rb:8
       400  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/time_with_zone.rb:149
       200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activemodel-5.0.1/lib/active_model/serialization.rb:131
       200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activemodel-5.0.1/lib/active_model/type/helpers/time_value.rb:61
       200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/serialization.rb:12
       200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/serialization.rb:14
       200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/serialization.rb:15
       200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/date_and_time/zones.rb:33
       200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/try.rb:6
       200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/tzinfo-1.2.2/lib/tzinfo/time_or_datetime.rb:314
       200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/tzinfo-1.2.2/lib/tzinfo/transition_data_timezone_info.rb:115
       101  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:142
       100  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activemodel-5.0.1/lib/active_model/serialization.rb:127
       100  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activemodel-5.0.1/lib/active_model/serialization.rb:134
       100  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activemodel-5.0.1/lib/active_model/serialization.rb:137
       100  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activemodel-5.0.1/lib/active_model/type/string.rb:15
       100  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/aggregations.rb:23
       100  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/associations.rb:268
       100  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/attribute_set/builder.rb:19
       100  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/attribute_set/builder.rb:20
       100  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/attribute_set/builder.rb:33
       100  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/attribute_set/builder.rb:65
       100  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/core.rb:135
       100  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/core.rb:546
       100  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/persistence.rb:69
       100  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/result.rb:123
       100  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/hash/transform_values.rb:14
       100  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/try.rb:17
        17  /Users/bti/.rvm/gems/ruby-2.4.0/gems/rack-2.0.1/lib/rack/utils.rb:469
        12  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_dispatch/http/response.rb:179
        12  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/connection_adapters/postgresql_adapter.rb:432
        11  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_dispatch/http/response.rb:412
         9  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/subscriber.rb:94
         9  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/tagged_logging.rb:54
         8  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/callbacks.rb:90
         7  /Users/bti/.rvm/rubies/ruby-2.4.0/lib/ruby/2.4.0/securerandom.rb:242
         6  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/inflector/methods.rb:69

allocated objects by class
-----------------------------------
      5218  String
      5208  Array
      2600  Rational
      1600  ActiveSupport::JSON::Encoding::JSONGemEncoder::EscapedString
      1255  Hash
       610  Time
       400  ActiveRecord::Attribute::FromDatabase
       206  MatchData
       200  TZInfo::TimeOrDateTime
       200  TZInfo::TimezonePeriod
       100  ActiveRecord::AttributeSet
       100  ActiveRecord::LazyAttributeHash
       100  User
        29  Proc
         7  ActionDispatch::Request
         5  ActiveSupport::Callbacks::Filters::Environment
         4  Rack::BodyProxy
         3  ActionDispatch::Response::Buffer
         3  ActionDispatch::Response::ContentTypeHeader
         3  ActiveSupport::HashWithIndifferentAccess
         3  ActiveSupport::Notifications::Event
         3  Rack::Utils::HeaderHash
         2  <<Unknown>>
         2  ActiveRecord::Result
         2  Digest::SHA256
         2  PG::Result
         2  StringIO
         1  ActionDispatch::Http::Headers
         1  ActionDispatch::Http::ParameterFilter
         1  ActionDispatch::Http::ParameterFilter::CompiledFilter
         1  ActionDispatch::Journey::Path::Pattern::MatchData
         1  ActionDispatch::RemoteIp::GetIp
         1  ActionDispatch::Response
         1  ActionDispatch::Response::Header
         1  ActionDispatch::Response::RackBody
         1  ActionView::LookupContext
         1  ActionView::PathSet
         1  ActiveRecord::Associations::Preloader
         1  ActiveRecord::ConnectionAdapters::AbstractAdapter::SQLString
         1  ActiveRecord::Relation
         1  ActiveSupport::ArrayInquirer
         1  ActiveSupport::Cache::Strategy::LocalCache::LocalStore
         1  ActiveSupport::JSON::Encoding::JSONGemEncoder
         1  Arel::Attributes::Attribute
         1  Arel::Nodes::JoinSource
         1  Arel::Nodes::SelectCore
         1  Arel::Nodes::SelectStatement
         1  Arel::Nodes::SqlLiteral
         1  Arel::SelectManager
         1  Class

retained memory by gem
-----------------------------------
NO DATA

retained memory by file
-----------------------------------
NO DATA

retained memory by location
-----------------------------------
NO DATA

retained memory by class
-----------------------------------
NO DATA

retained objects by gem
-----------------------------------
NO DATA

retained objects by file
-----------------------------------
NO DATA

retained objects by location
-----------------------------------
NO DATA

retained objects by class
-----------------------------------
NO DATA


Allocated String Report
-----------------------------------
       400  "acts_like_time?"
       400  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/acts_like.rb:8

       400  "time"
       400  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/acts_like.rb:8

       200  "\"created_at\""
       100  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34
       100  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55

       200  "\"id\""
       100  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34
       100  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55

       200  "\"name\""
       100  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34
       100  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55

       200  "\"updated_at\""
       100  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34
       100  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55

       200  "01"
       200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activemodel-5.0.1/lib/active_model/type/helpers/time_value.rb:71

       200  "03"
       200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activemodel-5.0.1/lib/active_model/type/helpers/time_value.rb:71

       200  "13"
       200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activemodel-5.0.1/lib/active_model/type/helpers/time_value.rb:71

       200  "2017"
       200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activemodel-5.0.1/lib/active_model/type/helpers/time_value.rb:71

       200  "21"
       200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activemodel-5.0.1/lib/active_model/type/helpers/time_value.rb:71

       200  "22"
       200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activemodel-5.0.1/lib/active_model/type/helpers/time_value.rb:71

        15  ""
         4  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/connection_adapters/postgresql_adapter.rb:427
         3  /Users/bti/.rvm/rubies/ruby-2.4.0/lib/ruby/2.4.0/logger.rb:598
         2  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_dispatch/http/request.rb:228
         1  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_dispatch/http/request.rb:141
         1  /Users/bti/.rvm/gems/ruby-2.4.0/gems/rack-2.0.1/lib/rack/mock.rb:102
         1  /Users/bti/.rvm/gems/ruby-2.4.0/gems/rack-2.0.1/lib/rack/mock.rb:107
         1  /Users/bti/.rvm/gems/ruby-2.4.0/gems/rack-2.0.1/lib/rack/mock.rb:131
         1  /Users/bti/.rvm/gems/ruby-2.4.0/gems/rack-2.0.1/lib/rack/mock.rb:97
         1  /Users/bti/.rvm/gems/ruby-2.4.0/gems/railties-5.0.1/lib/rails/rack/logger.rb:52

         7  "json"
         1  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_controller/log_subscriber.rb:11
         1  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_controller/metal/renderers.rb:91
         1  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_dispatch/http/mime_type.rb:174
         1  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_dispatch/journey/gtg/simulator.rb:32
         1  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_dispatch/journey/gtg/transition_table.rb:49
         1  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_dispatch/journey/path/pattern.rb:132
         1  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_dispatch/journey/router/utils.rb:58

         6  "\"2017-01-22T13:03:21.059Z\""
         4  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
         2  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

         6  "\"2017-01-22T13:03:21.064Z\""
         4  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
         2  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

         6  "\"2017-01-22T13:03:21.066Z\""
         4  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
         2  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

         6  "\"2017-01-22T13:03:21.069Z\""
         4  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
         2  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

         6  "\"2017-01-22T13:03:21.071Z\""
         4  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
         2  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

         6  "\"2017-01-22T13:03:21.073Z\""
         4  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
         2  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

         6  "\"2017-01-22T13:03:21.075Z\""
         4  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
         2  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

         6  "\"2017-01-22T13:03:21.079Z\""
         4  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
         2  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

         6  "\"2017-01-22T13:03:21.081Z\""
         4  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
         2  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

         6  "\"2017-01-22T13:03:21.083Z\""
         4  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
         2  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

         6  "\"2017-01-22T13:03:21.086Z\""
         4  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
         2  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

         6  "\"2017-01-22T13:03:21.088Z\""
         4  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
         2  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

         6  "\"2017-01-22T13:03:21.090Z\""
         4  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
         2  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

         6  "\"2017-01-22T13:03:21.093Z\""
         4  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
         2  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

         6  "\"2017-01-22T13:03:21.096Z\""
         4  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
         2  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

         6  "\"2017-01-22T13:03:21.098Z\""
         4  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
         2  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

         6  "\"2017-01-22T13:03:21.100Z\""
         4  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
         2  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

         6  "\"2017-01-22T13:03:21.102Z\""
         4  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
         2  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

         6  "\"2017-01-22T13:03:21.105Z\""
         4  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
         2  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

         6  "\"2017-01-22T13:03:21.108Z\""
         4  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
         2  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

         6  "\"2017-01-22T13:03:21.111Z\""
         4  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
         2  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

         6  "\"2017-01-22T13:03:21.113Z\""
         4  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
         2  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

         6  "\"2017-01-22T13:03:21.116Z\""
         4  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
         2  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

         6  "\"2017-01-22T13:03:21.118Z\""
         4  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
         2  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

         6  "\"2017-01-22T13:03:21.120Z\""
         4  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
         2  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

         6  "\"2017-01-22T13:03:21.123Z\""
         4  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
         2  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

         6  "\"2017-01-22T13:03:21.125Z\""
         4  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
         2  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

         6  "\"2017-01-22T13:03:21.128Z\""
         4  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
         2  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

         6  "\"2017-01-22T13:03:21.130Z\""
         4  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
         2  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

         6  "\"2017-01-22T13:03:21.133Z\""
         4  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
         2  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

         6  "\"2017-01-22T13:03:21.135Z\""
         4  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
         2  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

         6  "\"2017-01-22T13:03:21.137Z\""
         4  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
         2  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

         6  "\"2017-01-22T13:03:21.140Z\""
         4  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
         2  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

         6  "\"2017-01-22T13:03:21.142Z\""
         4  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
         2  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

         6  "\"2017-01-22T13:03:21.145Z\""
         4  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
         2  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

         6  "\"2017-01-22T13:03:21.148Z\""
         4  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
         2  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34


Retained String Report
----------------------------------
```


## Result with only 100 invocation

```
$ TEST_COUNT=100 PATH_TO_HIT='http://localhost:3000/users.json' bundle exec derailed exec perf:objects                                                                 [14:35:43]
/Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/xml_mini.rb:51: warning: constant ::Fixnum is deprecated
/Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/xml_mini.rb:52: warning: constant ::Bignum is deprecated
Booting: production
/Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/numeric/conversions.rb:138: warning: constant ::Fixnum is deprecated
/Users/bti/.rvm/gems/ruby-2.4.0/gems/activejob-5.0.1/lib/active_job/arguments.rb:38: warning: constant ::Fixnum is deprecated
/Users/bti/.rvm/gems/ruby-2.4.0/gems/activejob-5.0.1/lib/active_job/arguments.rb:38: warning: constant ::Bignum is deprecated
Endpoint: "http://localhost:3000/users.json"
/Users/bti/.rvm/gems/ruby-2.4.0/gems/derailed_benchmarks-1.3.2/lib/derailed_benchmarks/tasks.rb:72: warning: already initialized constant DERAILED_APP
/Users/bti/.rvm/gems/ruby-2.4.0/gems/derailed_benchmarks-1.3.2/lib/derailed_benchmarks/tasks.rb:23: warning: previous definition of DERAILED_APP was here
Running 100 times
Total allocated: 108092216 bytes (1790000 objects)
Total retained:  0 bytes (0 objects)

allocated memory by gem
-----------------------------------
  46036000  activesupport-5.0.1
  24568000  activerecord-5.0.1
  24297600  activemodel-5.0.1
   7870900  ruby-2.4.0/lib
   2880000  tzinfo-1.2.2
   1352016  actionpack-5.0.1
    741000  rack-2.0.1
    149100  railties-5.0.1
     91200  arel-7.1.4
     56000  actionview-5.0.1
     27200  string_allocation/app
     23200  concurrent-ruby-1.0.4

allocated memory by file
-----------------------------------
  18457600  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activemodel-5.0.1/lib/active_model/type/helpers/time_value.rb
  15946400  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb
  11520000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/time_with_zone.rb
   8404000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb
   8400000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/attribute_set/builder.rb
   7493800  /Users/bti/.rvm/rubies/ruby-2.4.0/lib/ruby/2.4.0/json/common.rb
   5440000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activemodel-5.0.1/lib/active_model/serialization.rb
   3200000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/attribute.rb
   3200000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/acts_like.rb
   2720000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/try.rb
   2400000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/serialization.rb
   2064000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/result.rb
   1970400  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/connection_adapters/postgresql/database_statements.rb
   1920000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/persistence.rb
   1920000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/hash/transform_values.rb
   1688000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/core.rb
   1600000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/attribute_methods/time_zone_conversion.rb
   1440000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/date_and_time/zones.rb
   1440000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/tzinfo-1.2.2/lib/tzinfo/time_or_datetime.rb
   1440000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/tzinfo-1.2.2/lib/tzinfo/transition_data_timezone_info.rb
    400000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activemodel-5.0.1/lib/active_model/type/string.rb
    400000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/aggregations.rb
    400000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/associations.rb
    308000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_dispatch/http/response.rb
    296800  /Users/bti/.rvm/gems/ruby-2.4.0/gems/rack-2.0.1/lib/rack/mock.rb
    269600  /Users/bti/.rvm/gems/ruby-2.4.0/gems/rack-2.0.1/lib/rack/utils.rb
    197400  /Users/bti/.rvm/rubies/ruby-2.4.0/lib/ruby/2.4.0/logger.rb
    180800  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/tagged_logging.rb
    170400  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/querying.rb
    156000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_dispatch/http/parameter_filter.rb
    134700  /Users/bti/.rvm/gems/ruby-2.4.0/gems/railties-5.0.1/lib/rails/rack/logger.rb
    134400  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/callbacks.rb
    122400  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/subscriber.rb
    120800  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_controller/metal/instrumentation.rb
    117600  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/notifications/instrumenter.rb
    104000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/connection_adapters/postgresql_adapter.rb
     90400  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/hash_with_indifferent_access.rb
     88000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_dispatch/journey/gtg/transition_table.rb
     86400  /Users/bti/.rvm/rubies/ruby-2.4.0/lib/ruby/2.4.0/uri/rfc2396_parser.rb
     84000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_controller/log_subscriber.rb
     82600  /Users/bti/.rvm/gems/ruby-2.4.0/gems/rack-2.0.1/lib/rack/etag.rb
     68000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_dispatch/http/mime_negotiation.rb
     66400  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_dispatch/journey/router.rb
     59200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_dispatch/journey/gtg/simulator.rb
     55700  /Users/bti/.rvm/rubies/ruby-2.4.0/lib/ruby/2.4.0/securerandom.rb
     52800  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/connection_adapters/abstract_adapter.rb
     52000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/rack-2.0.1/lib/rack/request.rb
     48000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/inflector/methods.rb
     44000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_dispatch/http/request.rb
     44000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_dispatch/journey/path/pattern.rb

allocated memory by location
-----------------------------------
   9920000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/time_with_zone.rb:481
   7471400  /Users/bti/.rvm/rubies/ruby-2.4.0/lib/ruby/2.4.0/json/common.rb:224
   7120000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:89
   6937600  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activemodel-5.0.1/lib/active_model/type/helpers/time_value.rb:69
   6400000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34
   4800000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activemodel-5.0.1/lib/active_model/type/helpers/time_value.rb:70
   4800000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activemodel-5.0.1/lib/active_model/type/helpers/time_value.rb:71
   4560000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:85
   4140000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
   3280000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/attribute_set/builder.rb:64
   3200000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/attribute.rb:5
   3200000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/acts_like.rb:8
   2400000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activemodel-5.0.1/lib/active_model/serialization.rb:131
   2004000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:142
   1931200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/connection_adapters/postgresql/database_statements.rb:111
   1920000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activemodel-5.0.1/lib/active_model/serialization.rb:134
   1920000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activemodel-5.0.1/lib/active_model/type/helpers/time_value.rb:61
   1920000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/attribute_set/builder.rb:33
   1920000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/attribute_set/builder.rb:65
   1920000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/persistence.rb:69
   1920000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/result.rb:123
   1920000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/hash/transform_values.rb:14
   1920000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/try.rb:17
   1600000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/attribute_methods/time_zone_conversion.rb:8
   1600000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/time_with_zone.rb:149
   1440000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/date_and_time/zones.rb:33
   1440000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/tzinfo-1.2.2/lib/tzinfo/time_or_datetime.rb:314
   1440000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/tzinfo-1.2.2/lib/tzinfo/transition_data_timezone_info.rb:115
   1280000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/core.rb:135
    880000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/attribute_set/builder.rb:19
    800000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/serialization.rb:12
    800000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/serialization.rb:14
    800000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/serialization.rb:15
    800000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/try.rb:6
    720000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activemodel-5.0.1/lib/active_model/serialization.rb:127
    400000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activemodel-5.0.1/lib/active_model/serialization.rb:137
    400000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activemodel-5.0.1/lib/active_model/type/string.rb:15
    400000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/aggregations.rb:23
    400000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/associations.rb:268
    400000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/attribute_set/builder.rb:20
    400000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/core.rb:546
    176000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/rack-2.0.1/lib/rack/mock.rb:92
    106400  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/querying.rb:50
    105600  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/notifications/instrumenter.rb:58
     96800  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_dispatch/http/response.rb:412
     88800  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/tagged_logging.rb:54
     84800  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_dispatch/http/parameter_filter.rb:40
     84300  /Users/bti/.rvm/rubies/ruby-2.4.0/lib/ruby/2.4.0/logger.rb:695
     84000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/result.rb:118
     84000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:91

allocated memory by class
-----------------------------------
  26665600  Array
  24344200  String
  19491200  Hash
  10400000  Rational
   6400000  ActiveSupport::JSON::Encoding::JSONGemEncoder::EscapedString
   5768000  MatchData
   5376000  Time
   3200000  ActiveRecord::Attribute::FromDatabase
   1440000  TZInfo::TimeOrDateTime
   1440000  TZInfo::TimezonePeriod
   1280000  User
    880000  ActiveRecord::LazyAttributeHash
    400000  ActiveRecord::AttributeSet
    232000  Proc
    101600  ActionDispatch::Request
    100800  Rack::Utils::HeaderHash
     57600  ActiveSupport::HashWithIndifferentAccess
     51200  Regexp
     45600  Class
     36000  ActiveSupport::Callbacks::Filters::Environment
     28800  ActiveSupport::Notifications::Event
     27200  StringScanner
     23200  <<Unknown>>
     22400  JSON::Ext::Generator::State
     21600  ActionDispatch::Response::Buffer
     17600  Thread::Mutex
     16000  Rack::BodyProxy
     16000  StringIO
     14400  ActiveRecord::Result
     13600  ActionDispatch::Response
     12016  UsersController
     12000  ActionDispatch::Response::ContentTypeHeader
     12000  URI::HTTP
     11200  ActiveRecord::Relation
     10400  Arel::Nodes::SelectCore
     10400  Rack::MockResponse
      8800  ActionView::LookupContext
      8800  Arel::Nodes::SelectStatement
      8000  Digest::SHA256
      8000  PG::Result
      4000  ActionDispatch::Http::Headers
      4000  ActionDispatch::Http::ParameterFilter
      4000  ActionDispatch::Http::ParameterFilter::CompiledFilter
      4000  ActionDispatch::Journey::Path::Pattern::MatchData
      4000  ActionDispatch::RemoteIp::GetIp
      4000  ActionDispatch::Response::Header
      4000  ActionDispatch::Response::RackBody
      4000  ActionView::PathSet
      4000  ActiveRecord::Associations::Preloader
      4000  ActiveRecord::ConnectionAdapters::AbstractAdapter::SQLString

allocated objects by gem
-----------------------------------
    861300  activesupport-5.0.1
    360000  activemodel-5.0.1
    339300  activerecord-5.0.1
    154800  ruby-2.4.0/lib
     40000  tzinfo-1.2.2
     20900  actionpack-5.0.1
      8500  rack-2.0.1
      2000  arel-7.1.4
      1800  railties-5.0.1
       900  actionview-5.0.1
       300  string_allocation/app
       200  concurrent-ruby-1.0.4

allocated objects by file
-----------------------------------
    300000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activemodel-5.0.1/lib/active_model/type/helpers/time_value.rb
    280400  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb
    260000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/time_with_zone.rb
    170100  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb
    150200  /Users/bti/.rvm/rubies/ruby-2.4.0/lib/ruby/2.4.0/json/common.rb
     90000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/attribute_set/builder.rb
     80000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/acts_like.rb
     60000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/serialization.rb
     50000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activemodel-5.0.1/lib/active_model/serialization.rb
     40800  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/connection_adapters/postgresql/database_statements.rb
     40000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/attribute.rb
     40000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/attribute_methods/time_zone_conversion.rb
     30000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/try.rb
     20200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/core.rb
     20000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/date_and_time/zones.rb
     20000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/tzinfo-1.2.2/lib/tzinfo/time_or_datetime.rb
     20000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/tzinfo-1.2.2/lib/tzinfo/transition_data_timezone_info.rb
     10900  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/result.rb
     10000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activemodel-5.0.1/lib/active_model/type/string.rb
     10000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/aggregations.rb
     10000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/associations.rb
     10000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/persistence.rb
     10000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/hash/transform_values.rb
      4900  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_dispatch/http/response.rb
      2800  /Users/bti/.rvm/gems/ruby-2.4.0/gems/rack-2.0.1/lib/rack/utils.rb
      2200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/connection_adapters/postgresql_adapter.rb
      2100  /Users/bti/.rvm/rubies/ruby-2.4.0/lib/ruby/2.4.0/logger.rb
      2000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/tagged_logging.rb
      2000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/rack-2.0.1/lib/rack/mock.rb
      1800  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/subscriber.rb
      1700  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_dispatch/http/parameter_filter.rb
      1700  /Users/bti/.rvm/gems/ruby-2.4.0/gems/railties-5.0.1/lib/rails/rack/logger.rb
      1600  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_dispatch/journey/gtg/transition_table.rb
      1600  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/callbacks.rb
      1500  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/hash_with_indifferent_access.rb
      1500  /Users/bti/.rvm/gems/ruby-2.4.0/gems/rack-2.0.1/lib/rack/etag.rb
      1300  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_controller/metal/instrumentation.rb
      1300  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_dispatch/http/mime_negotiation.rb
      1300  /Users/bti/.rvm/gems/ruby-2.4.0/gems/rack-2.0.1/lib/rack/request.rb
      1200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_controller/log_subscriber.rb
      1000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/numeric/conversions.rb
      1000  /Users/bti/.rvm/rubies/ruby-2.4.0/lib/ruby/2.4.0/uri/rfc2396_parser.rb
       900  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_dispatch/http/request.rb
       900  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_dispatch/journey/gtg/simulator.rb
       900  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_dispatch/journey/router.rb
       900  /Users/bti/.rvm/rubies/ruby-2.4.0/lib/ruby/2.4.0/securerandom.rb
       800  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_controller/metal/rendering.rb
       700  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/connection_adapters/abstract_adapter.rb
       700  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/querying.rb
       700  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/relation/query_methods.rb

allocated objects by location
-----------------------------------
    220000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/time_with_zone.rb:481
    160000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34
    150100  /Users/bti/.rvm/rubies/ruby-2.4.0/lib/ruby/2.4.0/json/common.rb:224
    120000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activemodel-5.0.1/lib/active_model/type/helpers/time_value.rb:70
    120000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activemodel-5.0.1/lib/active_model/type/helpers/time_value.rb:71
    100000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:89
     90000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
     90000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:85
     80000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/acts_like.rb:8
     50000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/attribute_set/builder.rb:64
     40200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/connection_adapters/postgresql/database_statements.rb:111
     40000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activemodel-5.0.1/lib/active_model/type/helpers/time_value.rb:69
     40000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/attribute.rb:5
     40000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/attribute_methods/time_zone_conversion.rb:8
     40000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/time_with_zone.rb:149
     20000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activemodel-5.0.1/lib/active_model/serialization.rb:131
     20000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activemodel-5.0.1/lib/active_model/type/helpers/time_value.rb:61
     20000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/serialization.rb:12
     20000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/serialization.rb:14
     20000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/serialization.rb:15
     20000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/date_and_time/zones.rb:33
     20000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/try.rb:6
     20000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/tzinfo-1.2.2/lib/tzinfo/time_or_datetime.rb:314
     20000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/tzinfo-1.2.2/lib/tzinfo/transition_data_timezone_info.rb:115
     10100  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:142
     10000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activemodel-5.0.1/lib/active_model/serialization.rb:127
     10000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activemodel-5.0.1/lib/active_model/serialization.rb:134
     10000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activemodel-5.0.1/lib/active_model/serialization.rb:137
     10000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activemodel-5.0.1/lib/active_model/type/string.rb:15
     10000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/aggregations.rb:23
     10000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/associations.rb:268
     10000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/attribute_set/builder.rb:19
     10000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/attribute_set/builder.rb:20
     10000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/attribute_set/builder.rb:33
     10000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/attribute_set/builder.rb:65
     10000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/core.rb:135
     10000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/core.rb:546
     10000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/persistence.rb:69
     10000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/result.rb:123
     10000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/hash/transform_values.rb:14
     10000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/try.rb:17
      1700  /Users/bti/.rvm/gems/ruby-2.4.0/gems/rack-2.0.1/lib/rack/utils.rb:469
      1200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_dispatch/http/response.rb:179
      1200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/connection_adapters/postgresql_adapter.rb:432
      1100  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_dispatch/http/response.rb:412
       900  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/subscriber.rb:94
       900  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/tagged_logging.rb:54
       800  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/callbacks.rb:90
       700  /Users/bti/.rvm/rubies/ruby-2.4.0/lib/ruby/2.4.0/securerandom.rb:242
       600  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/inflector/methods.rb:69

allocated objects by class
-----------------------------------
    521800  String
    520800  Array
    260000  Rational
    160000  ActiveSupport::JSON::Encoding::JSONGemEncoder::EscapedString
    125500  Hash
     61000  Time
     40000  ActiveRecord::Attribute::FromDatabase
     20600  MatchData
     20000  TZInfo::TimeOrDateTime
     20000  TZInfo::TimezonePeriod
     10000  ActiveRecord::AttributeSet
     10000  ActiveRecord::LazyAttributeHash
     10000  User
      2900  Proc
       700  ActionDispatch::Request
       500  ActiveSupport::Callbacks::Filters::Environment
       400  Rack::BodyProxy
       300  ActionDispatch::Response::Buffer
       300  ActionDispatch::Response::ContentTypeHeader
       300  ActiveSupport::HashWithIndifferentAccess
       300  ActiveSupport::Notifications::Event
       300  Rack::Utils::HeaderHash
       200  <<Unknown>>
       200  ActiveRecord::Result
       200  Digest::SHA256
       200  PG::Result
       200  StringIO
       100  ActionDispatch::Http::Headers
       100  ActionDispatch::Http::ParameterFilter
       100  ActionDispatch::Http::ParameterFilter::CompiledFilter
       100  ActionDispatch::Journey::Path::Pattern::MatchData
       100  ActionDispatch::RemoteIp::GetIp
       100  ActionDispatch::Response
       100  ActionDispatch::Response::Header
       100  ActionDispatch::Response::RackBody
       100  ActionView::LookupContext
       100  ActionView::PathSet
       100  ActiveRecord::Associations::Preloader
       100  ActiveRecord::ConnectionAdapters::AbstractAdapter::SQLString
       100  ActiveRecord::Relation
       100  ActiveSupport::ArrayInquirer
       100  ActiveSupport::Cache::Strategy::LocalCache::LocalStore
       100  ActiveSupport::JSON::Encoding::JSONGemEncoder
       100  Arel::Attributes::Attribute
       100  Arel::Nodes::JoinSource
       100  Arel::Nodes::SelectCore
       100  Arel::Nodes::SelectStatement
       100  Arel::Nodes::SqlLiteral
       100  Arel::SelectManager
       100  Class

retained memory by gem
-----------------------------------
NO DATA

retained memory by file
-----------------------------------
NO DATA

retained memory by location
-----------------------------------
NO DATA

retained memory by class
-----------------------------------
NO DATA

retained objects by gem
-----------------------------------
NO DATA

retained objects by file
-----------------------------------
NO DATA

retained objects by location
-----------------------------------
NO DATA

retained objects by class
-----------------------------------
NO DATA


Allocated String Report
-----------------------------------
     40000  "acts_like_time?"
     40000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/acts_like.rb:8

     40000  "time"
     40000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/acts_like.rb:8

     20000  "\"created_at\""
     10000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34
     10000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55

     20000  "\"id\""
     10000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34
     10000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55

     20000  "\"name\""
     10000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34
     10000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55

     20000  "\"updated_at\""
     10000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34
     10000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55

     20000  "01"
     20000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activemodel-5.0.1/lib/active_model/type/helpers/time_value.rb:71

     20000  "03"
     20000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activemodel-5.0.1/lib/active_model/type/helpers/time_value.rb:71

     20000  "13"
     20000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activemodel-5.0.1/lib/active_model/type/helpers/time_value.rb:71

     20000  "2017"
     20000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activemodel-5.0.1/lib/active_model/type/helpers/time_value.rb:71

     20000  "21"
     20000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activemodel-5.0.1/lib/active_model/type/helpers/time_value.rb:71

     20000  "22"
     20000  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activemodel-5.0.1/lib/active_model/type/helpers/time_value.rb:71

      1500  ""
       400  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activerecord-5.0.1/lib/active_record/connection_adapters/postgresql_adapter.rb:427
       300  /Users/bti/.rvm/rubies/ruby-2.4.0/lib/ruby/2.4.0/logger.rb:598
       200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_dispatch/http/request.rb:228
       100  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_dispatch/http/request.rb:141
       100  /Users/bti/.rvm/gems/ruby-2.4.0/gems/rack-2.0.1/lib/rack/mock.rb:102
       100  /Users/bti/.rvm/gems/ruby-2.4.0/gems/rack-2.0.1/lib/rack/mock.rb:107
       100  /Users/bti/.rvm/gems/ruby-2.4.0/gems/rack-2.0.1/lib/rack/mock.rb:131
       100  /Users/bti/.rvm/gems/ruby-2.4.0/gems/rack-2.0.1/lib/rack/mock.rb:97
       100  /Users/bti/.rvm/gems/ruby-2.4.0/gems/railties-5.0.1/lib/rails/rack/logger.rb:52

       700  "json"
       100  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_controller/log_subscriber.rb:11
       100  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_controller/metal/renderers.rb:91
       100  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_dispatch/http/mime_type.rb:174
       100  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_dispatch/journey/gtg/simulator.rb:32
       100  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_dispatch/journey/gtg/transition_table.rb:49
       100  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_dispatch/journey/path/pattern.rb:132
       100  /Users/bti/.rvm/gems/ruby-2.4.0/gems/actionpack-5.0.1/lib/action_dispatch/journey/router/utils.rb:58

       600  "\"2017-01-22T13:03:21.059Z\""
       400  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
       200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

       600  "\"2017-01-22T13:03:21.064Z\""
       400  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
       200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

       600  "\"2017-01-22T13:03:21.066Z\""
       400  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
       200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

       600  "\"2017-01-22T13:03:21.069Z\""
       400  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
       200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

       600  "\"2017-01-22T13:03:21.071Z\""
       400  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
       200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

       600  "\"2017-01-22T13:03:21.073Z\""
       400  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
       200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

       600  "\"2017-01-22T13:03:21.075Z\""
       400  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
       200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

       600  "\"2017-01-22T13:03:21.079Z\""
       400  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
       200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

       600  "\"2017-01-22T13:03:21.081Z\""
       400  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
       200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

       600  "\"2017-01-22T13:03:21.083Z\""
       400  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
       200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

       600  "\"2017-01-22T13:03:21.086Z\""
       400  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
       200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

       600  "\"2017-01-22T13:03:21.088Z\""
       400  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
       200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

       600  "\"2017-01-22T13:03:21.090Z\""
       400  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
       200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

       600  "\"2017-01-22T13:03:21.093Z\""
       400  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
       200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

       600  "\"2017-01-22T13:03:21.096Z\""
       400  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
       200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

       600  "\"2017-01-22T13:03:21.098Z\""
       400  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
       200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

       600  "\"2017-01-22T13:03:21.100Z\""
       400  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
       200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

       600  "\"2017-01-22T13:03:21.102Z\""
       400  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
       200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

       600  "\"2017-01-22T13:03:21.105Z\""
       400  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
       200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

       600  "\"2017-01-22T13:03:21.108Z\""
       400  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
       200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

       600  "\"2017-01-22T13:03:21.111Z\""
       400  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
       200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

       600  "\"2017-01-22T13:03:21.113Z\""
       400  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
       200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

       600  "\"2017-01-22T13:03:21.116Z\""
       400  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
       200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

       600  "\"2017-01-22T13:03:21.118Z\""
       400  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
       200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

       600  "\"2017-01-22T13:03:21.120Z\""
       400  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
       200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

       600  "\"2017-01-22T13:03:21.123Z\""
       400  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
       200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

       600  "\"2017-01-22T13:03:21.125Z\""
       400  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
       200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

       600  "\"2017-01-22T13:03:21.128Z\""
       400  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
       200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

       600  "\"2017-01-22T13:03:21.130Z\""
       400  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
       200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

       600  "\"2017-01-22T13:03:21.133Z\""
       400  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
       200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

       600  "\"2017-01-22T13:03:21.135Z\""
       400  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
       200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

       600  "\"2017-01-22T13:03:21.137Z\""
       400  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
       200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

       600  "\"2017-01-22T13:03:21.140Z\""
       400  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
       200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

       600  "\"2017-01-22T13:03:21.142Z\""
       400  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
       200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

       600  "\"2017-01-22T13:03:21.145Z\""
       400  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
       200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34

       600  "\"2017-01-22T13:03:21.148Z\""
       400  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/json/encoding.rb:55
       200  /Users/bti/.rvm/gems/ruby-2.4.0/gems/activesupport-5.0.1/lib/active_support/core_ext/object/json.rb:34


Retained String Report
-----------------------------------

```
