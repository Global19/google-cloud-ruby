## Migrating to google-cloud-language 1.0

The 1.0 release of the google-cloud-language client is a significant upgrade
based on a next-gen code generator. If you have used earlier versions of this
library, there have been a number of significant changes that may require
updates to calling code. This document will describe the changes that have been
made, and what you need to do to update your usage.

To summarize:

 *  The library has been broken out into three libraries. The new gems
    `google-cloud-language-v1` and `google-cloud-language-v1beta2` contain the
    actual client classes for versions V1 and V1beta2 of the Natural Language
    service, and the gem `google-cloud-language` now simply provides a
    convenience wrapper. See [Library Structure](#library-structure) for more
    info.
 *  This library uses a new configuration mechanism giving you closer control
    over endpoint address, network timeouts, and retry. See
    [Client Configuration](#client-configuration) for more info. Furthermore,
    when creating a client object, you can customize its configuration in a
    block rather than passing arguments to the constructor. See
    [Creating Clients](#creating-clients) for more info.
 *  Previously, methods typically had at least one positional argument. Now,
    all arguments are keyword arguments. Additionally, you can pass a proto
    request object instead of separate arguments. See
    [Passing Arguments](#passing-arguments) for more info.
 *  Some classes have moved into different namespaces. See
    [Class Namespaces](#class-namespaces) for more info.

### Library Structure

Older 0.x releases of the `google-cloud-language` gem were all-in-one gems that
included potentially multiple clients for multiple versions of the Natural
Language service. The `Google::Cloud::Language.new()` factory method would
return you an instance of a `Google::Cloud::Language::V1::LanguageServiceClient`
object for the V1 version of the service, or a
`Google::Cloud::Language::V1beta2::LanguageServiceClient` object for the
V1beta2 version of the service. All these classes were defined in the same gem.

With the 1.0 release, the `google-cloud-language` gem still provides factory
methods for obtaining clients. (The method signatures will have changed. See
[Creating Clients](#creating-clients) for details.) However, the actual client
classes have been moved into separate gems, on per service version. The
`Google::Cloud::Language::V1::LanguageService::Client` class, along with its
helpers and data types, are now part of the `google-cloud-language-v1` gem.
Similarly, the `Google::Cloud::Language::V1beta2::LanguageService::Client`
class is part of the `google-cloud-language-v1beta2` gem. 

For normal usage, you can continue to install the `google-cloud-language` gem
and continue to use factory methods to create clients. This will remain the
easiest way to use the Natural Language client. However, you may alternatively
choose to install only one of the versioned gems. For example, if you know you
will only use `V1` of the service, you can install `google-cloud-language-v1`
by itself, and construct instances of the
`Google::Cloud::Language::V1::LanguageService::Client` client class directly.

### Client Configuration

In older releases, if you wanted to customize performance parameters or
low-level behavior of the client (such as credentials, timeouts, or
instrumentation), you would pass a variety of keyword arguments to the client
constructor. It was also extremely difficult to customize the default settings.

With the 1.0 release, a configuration interface provides access to these
parameters, both global defaults and per-client settings. For example, to set
global credentials and default timeout for all language V1 clients:

```
Google::Cloud::Language::V1::LanguageService::Client.configure do |config|
  config.credentials = "/path/to/credentials.json"
  config.timeout = 10_000
end
```

Individual RPCs can also be configured independently. For example, to set the
timeout for the `analyze_sentiment` call:

```
Google::Cloud::Language::V1::LanguageService::Client.configure do |config|
  config.rpcs.analyze_sentinment.timeout = 20_000
end
```

### Creating Clients

In older releases, to create a client object, you would use the
`Google::Cloud::Language.new` class method. Keyword arguments were available to
select a service version and to configure parameters such as credentials and
timeouts.

Wiht the 1.0 release, use the `Google::Cloud::Language.language_service` class
method to create a client object. You may select a service version using the
`:version` keyword argument. However, other configuration parameters should be
set in a configuration block when you create the client.

Old:
```
client = Google::Cloud::Language.new credentials: "/path/to/credentials.json"
```

New:
```
client = Google::Cloud::Language.language_service do |config|
  config.credentials = "/path/to/credentials.json"
end
```

The configuration block is optional. If you do not provide it, or you do not
set some configuration parameters, then the default configuration is used. See
[Client Configuration](#client-configuration).

### Passing Arguments

In older releases, certain required arguments would be passed as positional
arguments, while most optional arguments would be passed as keyword arguments.

With the 1.0 release, all RPC arguments are passed as keyword arguments,
regardless of whether they are required or optional. For example:

Old:
```
client = Google::Cloud::Language.new

document = {
  content: "I love API calls!",
  type: Google::Cloud::Language::V1::Document::Type::PLAIN_TEXT
}
encoding = Google:Cloud::Language::V1::EncodingType::UTF8

# Document is a positional argument, while encoding_type is a keyword argument.
response = client.analyze_sentiment document, encoding_type: encoding
```

New:
```
client = Google::Cloud::Language.new

document = {
  content: "I love API calls!",
  type: Google::Cloud::Language::V1::Document::Type::PLAIN_TEXT
}
encoding = Google:Cloud::Language::V1::EncodingType::UTF8

# Both document and encoding_type are keyword arguments.
response = client.analyze_sentiment document: document, encoding_type: encoding
```

In the 1.0 release, it is also possible to pass a request object, either
as a hash or as a protocol buffer.

New:
```
client = Google::Cloud::Language.new

request = Google::Cloud::Language::V1::AnalyzeSentimentRequest.new(
  document: {
    content: "I love API calls!",
    type: Google::Cloud::Language::V1::Document::Type::PLAIN_TEXT
  },
  encoding_type: Google:Cloud::Language::V1::EncodingType::UTF8
)

# Pass a request object as a positional argument:
response = client.analyze_sentiment request
```

Finally, in older releases, to provide call options, you would pass a
`Google::Gax::CallOptions` object with the `:options` keyword argument. In the
1.0 release, pass call options using a _second set_ of keyword arguments.

Old:
```
client = Google::Cloud::Language.new

document = {
  content: "I love API calls!",
  type: Google::Cloud::Language::V1::Document::Type::PLAIN_TEXT
}

options = Google::Gax::CallOptions.new timeout: 10_000

response = client.analyze_sentiment document, options: options
```

New:
```
client = Google::Cloud::Language.new

document = {
  content: "I love API calls!",
  type: Google::Cloud::Language::V1::Document::Type::PLAIN_TEXT
}
encoding = Google:Cloud::Language::V1::EncodingType::UTF8

# Use a hash to wrap the normal call arguments (or pass a request object), and
# then add further keyword arguments for the call options.
response = client.analyze_sentiment(
  { document: document, encoding_type: encoding },
  timeout: 10_000
)
```

### Class Namespaces

In older releases, the client object was of class
`Google::Cloud::Language::V1::LanguageServiceClient`.
In the 1.0 release, the client object is of class
`Google::Cloud::Language::V1::LanguageService::Client`.
Note that most users will use the `Google::Cloud::Language.language_service`
factory method to create instances of the client object, so you may not need to
reference the actual class directly.

In older releases, the credentials object was of class
`Google::Cloud::Language::V1::Credentials`.
In the 1.0 release, the credentials object is of class
`Google::Cloud::Language::V1::LanguageService::Credentials`.
Again, most users will not need to reference this class directly.