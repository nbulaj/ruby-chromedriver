# ruby-chromedriver

[![Docker Pulls](https://img.shields.io/docker/pulls/nbulai/ruby-chromedriver.svg)](https://hub.docker.com/r/nbulai/ruby-chromedriver/)
[![Docker Stars](https://img.shields.io/docker/stars/nbulai/ruby-chromedriver.svg)](https://hub.docker.com/r/nbulai/ruby-chromedriver/)

Ruby 2.6.x, Chrome / [Chrome driver](https://sites.google.com/a/chromium.org/chromedriver/), NodeJS on 
Docker for [Capybara](https://github.com/teamcapybara/capybara)/[Cucumber](https://github.com/cucumber/cucumber) specs.

Based on official Ruby 2.x image and uses official stable Chrome repository. Uses X virtual framebuffer (Xvfb)
for keycode conversions.

Can be used for Continuous Integration or Delivery (CI/CD) pipelines (like GitLab) instead of Qt and PhantomJS:

# CI/CD Configuration

Example of GitLab CI configuration for Ruby on Rails project with Cucumber using the image
(read https://about.gitlab.com/2017/12/19/moving-to-headless-chrome/):

```yaml
cucumber:
  stage: test
  image: 'nbulai/ruby-chromedriver:latest'
  variables:
    RAILS_ENV: test
  script:
    - chromedriver -v # to check version
    - ...
    - bin/cucumber
  artifacts:
    paths:
      - coverage/
  only:
    - master
```

If you need some specific screen size for testing, then you can pass it as an `SCREEN` environment variable:

`docker run -e SCREEN="1280x1024x16" -t -i --rm nbulai/ruby-chromedriver:latest bash`

Don't forget to configure your Cucumber/Capybara to use Chrome as a driver (see instruction below).

First install it locally to run your test-suite (if you will run tests on the dev machine):

```bash
# add Google public key
wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | apt-key add -
# add Google Chrome stable repository to sources
echo "deb http://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/google.list
sudo apt-get update 
# install Chrome and it's driver
sudo apt-get install google-chrome-stable chromium-chromedriver
sudo ln -s /usr/lib/chromium-browser/chromedriver /usr/bin/chromedriver
# check if all is OK
chromedriver -v
```

Add gem `selenium-webdriver` to your `Gemfile`:

```ruby
group :test do
  # ...
  gem 'selenium-webdriver'
end
```

And then edit `features/support/env.rb` (or other config file):

```ruby
# ...

Capybara.register_driver(:headless_chrome) do |app|
  capabilities = Selenium::WebDriver::Remote::Capabilities.chrome(
    # Add 'no-sandbox' arg if you have an "unknown error: Chrome failed to start: exited abnormally"
    # @see https://github.com/SeleniumHQ/selenium/issues/4961
    chromeOptions: { args: %w[headless disable-gpu no-sandbox] } 
  )

  Capybara::Selenium::Driver.new(app, browser: :chrome, desired_capabilities: capabilities)
end

Capybara.javascript_driver = :headless_chrome

# ...
```

That's all. Run `bin/cucumber` to check your tests.

## License

This repo content is released under the [MIT License](http://www.opensource.org/licenses/MIT).

Copyright (c) 2017-2019 Nikita Bulai (bulajnikita@gmail.com).
