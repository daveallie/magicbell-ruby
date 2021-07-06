# MagicBell Ruby Library

[MagicBell](https://magicbell.com) is the notification inbox for your web & mobile applications.

Please familiarize yourself with the [core concepts of MagicBell](https://magicbell.com/docs/core-concepts) before using this gem.

This gem:

1. Makes it easy to interact with [MagicBell's REST API](https://magicbell.com/docs/rest-api/overview) from Ruby. You can use it to create notifications in your project, fetch them, etc.
2. Helps you calculate the HMAC for a user's email or external_id when you turn on [HMAC Authentication](https://magicbell.com/docs/turn-on-hmac-authentication) for your project.

<img width="415" alt="MagicBell Notification Inbox" src="https://files.readme.io/c09b21a-image1.png">

## Installation

First, sign up at https://magicbell.com and grab your MagicBell project's API key and secret from the "Settings" section of your MagicBell dashboard.

Then, add the gem to your app's Gemfile:

```ruby
# Gemfile
gem "magicbell"
```

and install it:

```
bundle install
```

## Configuration

### Global configuration

The gem will automatically pick your MagicBell project's API key and secret from the `MAGICBELL_API_KEY` and `MAGICBELL_API_SECRET` environment variables, respectively.

Alternatively, you can configure your MagicBell manually. For example, for a rails project, create an initializer file for MagicBell and set your project's keys:

```ruby
# config/initializers/magicbell.rb

MagicBell.configure do |config|
  config.api_key = "MAGICBELL_API_KEY"
  config.api_secret = "MAGICBELL_API_SECRET"
end
```

### Per-client configuration

You may need to use a different API key and secret for each client if you integrate more than one MagicBell project into your application. To override the global configuration, provide the specific keys when you create instances of `MagicBell::Client`:

```ruby
magicbell = MagicBell::Client.new(
  api_key: "MAGICBELL_PROJECT_API_KEY",
  api_secret: "MAGICBELL_PROJECT_API_SECRET"
)
```

Keep in mind that instances of `MagicBell::Client` will default to the global configuration unless an API key and secret are provided.

## API wrapper

### Create a notification

You can send a notification to one or many users by identifying them by their email address:

```ruby
magicbell = MagicBell::Client.new
magicbell.create_notification(
  title: "Rob assigned a task to you",
  recipients: [
    { email: "joe@example.com" },
  ]
)
```

Or you can identify users by an `external_id` (their ID in your database, for example):

```ruby
magicbell = MagicBell::Client.new
magicbell.create_notification(
  title: "Rob assigned a task to you",
  recipients: [{
    external_id: "DATABASE_ID"
  }]
)
```

This method has the benefit of allowing users to access their notifications when their email address changes. Make sure you identify users by their `externalID` when you [initialize the notification inbox](https://magicbell.com/docs/react/identifying-users), too.

You can also provide content for the notification and a URL to redirect the user to when they click on the notification from the [notification inbox](https://magicbell.com/docs/adding-magicbell-to-your-product):

```ruby
magicbell = MagicBell::Client.new
magicbell.create_notification(
  title: "Rob assigned to a task to you",
  content: "Hey Joe, can give this customer a demo of our app?",
  action_url: "https://example.com/task_path",
  recipients: [
    { email: "joe@example.com" },
  ],
)
```

### Fetch a user's notifications

To fetch a user's notifications you can do this:

```ruby
magicbell = MagicBell::Client.new

user = magicbell.user_with_email("joe@example.com")
user.notifications.each { |notification| puts notification.attribute("title") }
```

Please note that the example above fetches the user's 15 most recent notifications (the default number per page). If you'd like to fetch subsequent pages, use the `each_page` method instead:

```ruby
magicbell = MagicBell::Client.new

user = magicbell.user_with_email("joe@example.com")
user.notifications.each_page do |page|
  page.each do |notification|
    puts notification.attribute("title")
  end
end
```

### Mark a user's notification as read/unread

```ruby
magicbell = MagicBell::Client.new

user = magicbell.user_with_email("joe@example.com")

notification = user.notifications.first
notification.mark_as_read
notification.mark_as_unread
```

### Mark all notifications of a user as read

```ruby
magicbell = MagicBell::Client.new

user = magicbell.user_with_email("joe@example.com")
user.mark_all_notifications_as_read
```

### Mark all notifications of a user as seen

```ruby
magicbell = MagicBell::Client.new

user = magicbell.user_with_email("joe@example.com")
user.mark_all_notifications_as_seen
```

### Error handling

This gem raises a `MagicBell::Client::HTTPError` if an API returns a non-2xx response.

```ruby
begin
  magicbell = MagicBell::Client.new
  magicbell.create_notification(
    title: "Rob assigned to a task to you"
  )
rescue MagicBell::Client::HTTPError => e
  # Report the error to your error tracker, for example
  error_context = {
    response_status: e.response_status,
    response_headers: e.response_headers,
    response_body: e.response_body
  }

  ErrorReporter.report(e, context: error_context)
end
```

## HMAC authentication

Please refer to our docs to know [how to turn on HMAC authentication](https://magicbell.com/docs/turn-on-hmac-authentication) for your MagicBell project.

### Calculate HMAC

This gem provides some helpers to calculate the HMAC for a user.

You can use the `MagicBell.hmac` method. Note that in this case, the API secret, which is used to generate the HMAC, will be fetched from the global configuration.

```ruby
hmac = MagicBell.hmac("joe@example.com")
```

You can also use the API secret of a specific client instance to calculate the HMAC:

```ruby
magicbell = MagicBell::Client.new(
  api_key: 'your_api_key',
  api_secret: 'your_api_secret'
)

hmac = magicbell.hmac("joe@example.com")
```

## API docs

Please visit [our website](https://magicbell.com) and [our docs](https://magicbell.com/docs) for more information.

## Contact Us

Have a query or hit upon a problem? Feel free to contact us at [hello@magicbell.io](mailto:hello@magicbell.io).
