# Heroku Lessons Learned

## Heroku with Rails running Unicorn

Unicorn is a web server for rails that runs concurrent processes allowing your
rails application to handle requests concurrently. It is currently the
recommended way of deploying a rails application.

Heroku's 1x dynos have a 512 MB memory cap and unicorn workers (especially if
many are being forked) have a tendency to breach this memory cap, generating
`H14` heroku warnings, and using swap space.

In order to solve this problem there is a handy gem that handles managing
unicorn worker memory.

Add `unicorn-worker-killer` to the production and/or staging gemsets.

```ruby
group :production do
  gem "unicorn-worker-killer"
end
```

Set unicorn workers to 2

```ruby
# config/unicorn.rb
worker_processes (ENV["UNICORN_WORKERS"] || 2).to_i
# ...
```

`config.ru` is what tells rack what to do. We need to tell it to let unicorn
worker killer manage the unicorn processes.

```ruby
if ENV['RAILS_ENV'] == 'production' || ENV['RAILS_ENV'] == 'staging'
  require 'unicorn/worker_killer'

  max_request_min =  500
  max_request_max =  600

  # Max requests per worker
  use Unicorn::WorkerKiller::MaxRequests, max_request_min, max_request_max

  oom_min = (ENV['OOM_MIN'].to_i || 240) * (1024**2)
  oom_max = (ENV['OOM_MAX'].to_i || 260) * (1024**2)

  # Max memory size (RSS) per worker
  use Unicorn::WorkerKiller::Oom, oom_min, oom_max
end

# rest of config.ru...
```

This has worked successfully for me and now i'm no longer getting H14 warning
and my memory usage is consistently below 500MB.

Success! :)

![Heroku Success](https://raw.githubusercontent.com/EnigmaCreativeGroup/lessons-learned/master/img/heroku-memory.png)

