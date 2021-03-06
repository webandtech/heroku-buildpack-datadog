Datadog Heroku Buildpack
========================

A [Heroku buildpack](https://devcenter.heroku.com/articles/buildpacks) to add [Datadog](https://www.datadoghq.com) to a Heroku Dyno.

## Usage

This buildpack installs the Datadog Agent in your Heroku Dyno, allowing you to collect system metrics, custom application metrics and traces. To collect custom application metrics or traces, you will also need to include the language appropriate [DogStatsD or Datadog APM library](http://docs.datadoghq.com/libraries/).

### Installation

To add this buildpack to your project, as well as setting the required environment variables:

```shell
cd <root of my project>

# If this is a new Heroku project
heroku create

# Add the appropriate language-specific buildpack. For example:
heroku buildpacks:add heroku/ruby

# Add this buildpack and set your environment variables
heroku buildpacks:add --index 1 https://github.com/DataDog/heroku-buildpack-datadog.git
heroku config:set DD_HOSTNAME=$(heroku apps:info|grep ===|cut -d' ' -f2)
heroku config:add DD_API_KEY=<your API key>

# Deploy to Heroku
git push heroku master
```

Once complete, the Datadog Agent will be started automatically when each Dyno starts.

The Datadog agent provides a listening port on 8125 for statsd/dogstatsd metrics and events. Traces are collected on port 8126.

### Configuration

In addition to the environment variables shown above, there are a number of others you can set:

| Setting | Description|
| --- | --- |
| DD_API_KEY | *Required.* Your API key is available from the [Datadog API integrations](https://app.datadoghq.com/account/settings#api) page. Note that this is the *API* key, not the application key. |
| DD_HOSTNAME | *Required.* Because Heroku Dynos are ephemeral and your application my be served by any available Dyno resource, you should set the hostname to your application or service name. This will give you more consistent metrics. To view metrics by Dyno hosts, the tag `dynohost` is added by the buildpack. |
| DD_TAGS | *Optional.* Sets additional tags provided as a comma-delimited string. For example, `heroku config:set DD_TAGS=simple-tag-0,tag-key-1:tag-value-1`. The buildpack automatically adds the tags `dyno` and `dynohost` which represent the Dyno name (e.g. web.1) and host ID (e.g. ) respectively. See the ["Guide to tagging"](http://docs.datadoghq.com/guides/tagging/) for more information. |
| DD_HISTOGRAM_PERCENTILES | *Optional.* You can optionally set additional percentiles for your histogram metrics. See [Histogram percentiles](#histogram-percentiles) below for more information. |
| DISABLE_DATADOG_AGENT | *Optional.* When set, the Datadog agent will not be run. |
| DD_AGENT_VERSION | *Optional.* By default, the buildpack will install the latest version of the Datadog Agent available in the package repository. Use this variable to install older versions of the Datadog Agent (note that not all versions of the Agent may be available). |
| DD_SERVICE_NAME | *Optional.* While not read directly by the Datadog Agent, we highly recommend that you set an environment variable for your service name. See the [Service Name](#service-name) section below for more information. |
| DD_SERVICE_ENV | *Optional.* The Datadog Agent will automatically try to identify your environment by searching for a tag in the form `env:<your environment name>`. If you do not set a tag or wish to override an existing tag, you can set the environment with this setting. For more information, see the [Datadog Tracing environments page](https://docs.datadoghq.com/tracing/environments/). |

### Histogram percentiles

You can optionally set additional percentiles for your histogram metrics. By default only the 95th percentile will be generated. To generate additional percentiles, set *all* percentiles, including the default, using the env variable `DD_HISTOGRAM_PERCENTILES`.  For example, if you want to generate 0.95 and 0.99 percentiles, you may use following command:

```shell
heroku config:add DD_HISTOGRAM_PERCENTILES="0.95, 0.99"
```

For more information about about additional percentiles, see the [percentiles documentation](https://help.datadoghq.com/hc/en-us/articles/204588979-How-to-graph-percentiles-in-Datadog).

### Service name

A service is a named set of processes that do the same job, such as `webapp` or `database`. The service name provides context when evaluating your trace data.

Although the service name is passed to Datadog on the application level, we highly recommend that you set the value as an environment variable, rather than directly in your application code.

For example, set your service name as an environment variable:

```shell
heroku config:set DD_SERVICE_NAME=my-webapp
```

Then in a python web application, you could set the service name from the environment variable:

```python
import os
from ddtrace import tracer

service_nane = os.environ.get('DD_SERVICE_NAME')
span = tracer.trace("web.request", service=service_name)
...
span.finish()
```

For Ruby on Rails applications, you'll need to configure the `config/initializers/datadog-tracer.rb` file:

```ruby
Rails.configuration.datadog_trace = {
  default_service: ENV['DD_SERVICE_NAME'] || 'my-app',
}
```

Setting the service name will vary according to your language or supported framework. Please reference the [Datadog libraries list](https://docs.datadoghq.com/libraries/) for specific language support.


## Contributing

This project is open source (Apache 2 License), which means we're happy for you to fork it, but we'd be even more excited to have you contribute back to it.

### Submitting issues

  * If you think you've found an issue, please search the [project issues](https://github.com/DataDog/heroku-buildpack-datadog/issues) and the [Troubleshooting](https://datadog.zendesk.com/hc/en-us/sections/200766955-Troubleshooting)
    section of our [Knowledge base](https://datadog.zendesk.com/hc/en-us) to see if it's known.
  * If you can't find anything useful, please contact our [support](http://docs.datadoghq.com/help/) and send a flare. To send a flare, you'll need get to your Dyno's command line:
  ```shell
  # From your project directory:
  heroku run bash

  # Once your Dyno has started and you are at the command line, send a flare:
  agent -c /app/.apt/etc/datadog-agent/datadog.yaml flare
  ```
  * Finally, you can open a Github issue.

### Pull requests

Have you fixed a bug or written a new check and want to share it? Many thanks!

In order to ease/speed up our review, here are some items you can check/improve when submitting your PR:

  * Keep it small and focused. Avoid changing too many things at once.
  * Summarize your PR with an explanatory title and a message describing your changes. Cross-reference any related bugs/PRs and provide steps for testing when appropriate.
  * Write meaningful commit messages. The commit message should describe the reason for the change and give extra details that will allow someone later on to understand in 5 seconds the thing you've been working on for a day.
  * Squash your commits. Please rebase your changes on master and squash your commits whenever possible, it keeps history cleaner and it's easier to revert things.

## History

Earlier versions of this project were forked from the [miketheman heroku-buildpack-datadog project](https://github.com/miketheman/heroku-buildpack-datadog). It was largely rewritten for Datadog's Agent version 6. Changes and more information can be found in the [changelog](https://github.com/DataDog/heroku-buildpack-datadog/blob/master/CHANGELOG.md).

To run the previous Agent-5-based version of this project, run the following from your project directory:
```shell
# Remove the old untagged buildpack
heroku buildpacks:remove https://github.com/DataDog/heroku-buildpack-datadog.git
# Add the tagged version of the buildpack
heroku buildpacks:add --index 1 https://github.com/DataDog/heroku-buildpack-datadog.git#legacy
```
