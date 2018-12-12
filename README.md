# ExFacebookMessenger
[![Build Status](https://travis-ci.org/oarrabi/facebook_messenger.svg?branch=master)](https://travis-ci.org/oarrabi/facebook_messenger)
[![Hex.pm](https://img.shields.io/hexpm/v/facebook_messenger.svg)](https://hex.pm/packages/facebook_messenger)
[![API Docs](https://img.shields.io/badge/api-docs-yellow.svg?style=flat)](http://hexdocs.pm/facebook_messenger/)
[![Coverage Status](https://coveralls.io/repos/github/oarrabi/facebook_messenger/badge.svg?branch=master)](https://coveralls.io/github/oarrabi/facebook_messenger?branch=master)
[![Inline docs](http://inch-ci.org/github/oarrabi/facebook_messenger.svg?branch=master)](http://inch-ci.org/github/oarrabi/facebook_messenger)

ExFacebookMessenger is a library that helps you create facebook messenger bots easily.

## Installation

```elixir
def deps do
  [{:facebook_messenger, git: "https://github.com/kirilkoroves/facebook_messenger.git", override: true}]
end
```


## Usage

### With plug
To create an echo back bot, do the following:

In your `Plug.Router` define a `forward` with a route to `FacebookMessenger.Router`

```elixir
defmodule Sample.Router do
  use Plug.Router
  ...

  forward "/messenger/webhook",
    to: FacebookMessenger.Router,
    message_received: &Sample.Router.message/1

  def message(msg) do
    message = FacebookMessenger.Response.parse(msg)
      |> FacebookMessenger.Response.get_messaging

    case message.type do
      "postback" -> YourApplication.process_postback(message)
      "message" -> YourApplication.proccess_text_message(message)
      _ -> YourApplication.handle_default(message)
    end
  end
end

defmodule YourApplication do
  def process_postback(message) do
    sender = FacebookMessenger.Response.message_senders(message) |> hd

    case message.payload do
      "USER_CLICKED_BUTTON" -> FacebookMessenger.Sender.send(sender, "You have clicked the button")
      _ -> FacebookMessenger.Sender.send(sender, "I can't handle this message")
    end
  end


  def process_text_message(message) do
    text = FacebookMessenger.Response.message_texts(message) |> hd
    sender = FacebookMessenger.Response.message_senders(message) |> hd
    FacebookMessenger.Sender.send(sender, text)
  end
end

```

This defines a webhook endpoint at:
`http://your-app-url/messenger/webhook`

Go to your `config/config.exs` and add the required configurations

```elixir
config :facebook_messenger,
      facebook_page_token: "Your facebook page token",
      challenge_verification_token: "the challenge verify token"
```

To get the `facebook_page_token` and `challenge_verification_token` follow the instructions [here ](https://developers.facebook.com/docs/messenger-platform/quickstart)