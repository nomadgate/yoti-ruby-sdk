# Yoti Ruby SDK

Welcome to the Yoti Ruby SDK. This repository contains the tools you need to quickly integrate your Ruby back-end with Yoti so that your users can share their identity details with your application in a secure and trusted way.

## Table of Contents

1) [An Architectural view](#an-architectural-view) -
High level overview of integration

2) [References](#references)-
Guides before you start

3) [Requirements](#requirements)-
Everything you need to get started

4) [Installing the SDK](#installing-the-sdk)-
How to install our SDK

5) [SDK Project import](#sdk-project-import)-
How to install the SDK to your project

6) [Configuration](#configuration)-
entry point explanation

7) [Profile Retrieval](#profile-retrieval)-
How to retrieve a Yoti profile using the token

8) [Handling users](#handling-users)-
How to manage users

9) [API Coverage](#api-coverage)-
Attributes defined

10) [Running the examples](running-the-examples)-
Attributes defined

11) [Support](#support)-
Please feel free to reach out

12) [Change Log](#change-log)

13) [License](#license)

## An Architectural view

To integrate your application with Yoti, your back-end must expose a GET endpoint that Yoti will use to forward tokens.
The endpoint can be configured in your Yoti Dashboard when you create/update your application. It can be found in the Integration section under the Callback URL name.

The image below shows how your application back-end and Yoti integrate into the context of a Login flow.
Yoti SDK carries out for you steps 6, 7, 8 and the profile decryption in step 9.

![alt text](login_flow.png "Login flow")


Yoti also allows you to enable user details verification from your mobile app by means of the Android (TBA) and iOS (TBA) SDKs. In that scenario, your Yoti-enabled mobile app is playing both the role of the browser and the Yoti app. Your back-end doesn't need to handle these cases in a significantly different way, but you might decide to handle the `User-Agent` header in order to provide different responses for desktop and mobile clients.

## References

* [AES-256 symmetric encryption][]
* [RSA pkcs asymmetric encryption][]
* [Protocol buffers][]
* [Base64 data][]

[AES-256 symmetric encryption]:   https://en.wikipedia.org/wiki/Advanced_Encryption_Standard
[RSA pkcs asymmetric encryption]: https://en.wikipedia.org/wiki/RSA_(cryptosystem)
[Protocol buffers]:               https://en.wikipedia.org/wiki/Protocol_Buffers
[Base64 data]:                    https://en.wikipedia.org/wiki/Base64

## Requirements

The Yoti gem requires at least Ruby 2.0.0.
If you're using a version of Ruby lower than 2.2.2 you might encounter issues when [Bundler][] tries to install the [Active Support][] gem. This can be avoided by manually requiring activesupport 4.2.

```ruby
gem activesupport '~> 4.2'
```

Versions of [Bundler][] > 1.13 will sort this dependency issue automatically. More info in this [comment][] by André Arko.

[comment]: https://github.com/bundler/bundler-features/issues/120#issuecomment-214514847
[Bundler]: http://bundler.io/
[Active Support]: https://rubygems.org/gems/activesupport/

## Installing the SDK

To import the Yoti SDK inside your project, add this line to your application's Gemfile:

```ruby
gem 'yoti'
```

And then execute:

```shell
$ bundle install
```

Or simply run the following command from your terminal:

```shell
$ [sudo] gem install yoti
```

## SDK Project import

The gem provides a generator for the initialization file:

```shell
$ rails generate yoti:install
```

The generated initialisation file can be found in `config/initializers/yoti.rb`.

## Configuration

A minimal Yoti client initialisation looks like this:

```ruby
Yoti.configure do |config|
  config.client_sdk_id = ENV['YOTI_CLIENT_SDK_ID']
  config.key_file_path = ENV['YOTI_KEY_FILE_PATH']
end
```
Make sure the following environment variables can be accessed by your app:

`YOTI_CLIENT_SDK_ID` - found on the Key settings page on your application dashboard

`YOTI_KEY_FILE_PATH` - the full path to your security key downloaded from the *Keys* settings page (e.g. /Users/developer/access-security.pem)

The following options are available:

Config               | Required | Default              | Note
---------------------|----------|----------------------|-----
`client_sdk_id`      | Yes      |                      | SDK identifier generated by when you publish your app
`key_file_path`      | Yes      |                      | Path to the pem file generated when you create your app
`api_url`            | No       | https://api.yoti.com | Path to Yoti URL used for debugging purposes
`api_port`           | No       | 443                  | Path to Yoti port used for debugging purposes

Keeping your settings and access keys outside your repository is highly recommended. You can use gems like [dotenv][] to manage environment variables more easily.

[dotenv]: https://github.com/bkeepers/dotenv

### Deploying to Heroku / AWS Elastic Beanstalk

Although we recommend using a pem file to store your secret key, and take advantage of the UNIX file permissions, your hosting provider might not allow access to the file system outside the deployment process.

If you're using Heroku or other alternative services, you can store the content of the secret key in an environment variable.

Your configuration should look like this:

```ruby
Yoti.configure do |config|
  config.client_sdk_id = ENV['YOTI_CLIENT_SDK_ID']
  config.key = ENV['YOTI_KEY']
end
```

Where `YOTI_KEY` is an environment variable with the following format:

```
YOTI_KEY="-----BEGIN RSA PRIVATE KEY-----\nMIIEp..."
```

An easier way of setting this on Heroku would be to use the [Heroku Command Line][]

```shell
heroku config:add YOTI_KEY ="$(cat your-access-security.pem)"
```

[Heroku Command Line]: https://devcenter.heroku.com/articles/heroku-command-line



## Profile retrieval

When your application receives a token via the exposed endpoint (it will be assigned to a query string parameter named `token`), you can easily retrieve the user profile:

```ruby
yoti_activity_details = Yoti::Client.get_activity_details(params[:token])
```

Before you inspect the user profile, you might want to check whether the user validation was successful. This is done as follows:

```ruby
if yoti_activity_details.outcome == 'SUCCESS'
  user_profile = yoti_activity_details.user_profile
else
  # handle unhappy path
end
```

The `user_profile ` object provides a set of attributes corresponding to user attributes. Whether the attributes are present or not depends on the settings you have applied to your app on Yoti Dashboard.

### Handling users

When you retrieve the user profile, you receive a user ID generated by Yoti exclusively for your application. This means that if the same individual logs into another app, Yoti will assign them a different id. You can use such id to verify whether the retrieved profile identifies a new or an existing user. Here is an example of how this works:

```ruby
if yoti_activity_details.outcome == 'SUCCESS'
  user = your_user_search_function(yoti_activity_details.user_id)

  if user
    # handle login
  else
    # handle registration
  end
else
  # handle unhappy path
end
```

Where `your_user_search_function` is a piece of logic in your app that is supposed to find a user, given a user_id. Regardless of wether the user is a new or an existing one, Yoti will always provide their profile, so you don't necessarily need to store it.

## Running the examples

The examples can be found in the [examples folder](examples).
For them to work you will need a working callback URL that your browser can redirect to. A good way of doing this is to use [ngrok][] to expose the local development URL. The callback URL for both examples will be: `http://your-local-url.domain/profile`.

The examples also use the `YOTI_APPLICATION_ID` environment variable to display the Yoti Connect button. This value can be found in your Yoti account, on the *Keys* settings page.

### Ruby on Rails

* rename the [.env.default](examples/rails/.env.default) file to `.env` and fill in the required configuration values
* install the dependencies with `bundle install`
* start the server `rails server`

Visiting the `http://your-local-url.domain` should show a Yoti Connect button

### Sinatra

* rename the [.env.default](examples/sinatra/.env.default) file to `.env` and fill in the required configuration values
* install the dependencies with `bundle install`
* start the server `dotenv ./app.rb`

Visiting the `http://your-local-url.domain` should show a Yoti Connect button

[ngrok]:          https://ngrok.com/

## API coverage

* Activity Details
    * [X] User ID `user_id`
    * [X] Profile
        * [X] Photo `selfie`
        * [X] Given Names `given_names`
        * [X] Family Name `family_name`
        * [X] Mobile Number `phone_number`
        * [X] Email address `email_address`
        * [X] Date of Birth `date_of_birth`
        * [X] Address `postal_address`
        * [X] Gender `gender`
        * [X] Nationality `nationality`

## Support

For any questions or support please email [sdksupport@yoti.com](mailto:sdksupport@yoti.com).
Please provide the following the get you up and working as quick as possible:

- Computer Type
- OS Version
- Version of Ruby being used
- Screenshot


## Changelog

See recent changes in the [release notes][release notes] or the [changelog](CHANGELOG.md).

[release notes]: https://github.com/getyoti/ruby/releases

## License

The gem is available under the following [terms](LICENSE.txt).
