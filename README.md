# ruby-chromedriver

Ruby 2.4, Chrome / Chrome driver, NodeJS on Docker for Capybara/Cucumber specs.

Based on official Ruby 2.4 image and uses stable Chrome official repository. Uses X virtual framebuffer (Xvfb) for keycode conversions.

Can be used for Continuous Integration or Delivery (CI/CD) pipelines (like GitLab) instead of Qt and PhantomJS:

```yaml
cucumber:
  stage: test
  image: 'gitlab.your-company-domain.com/group/project/ruby-chromedriver:2.32'
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

Don't forget to configure your Cucumber/Capybara to use Chrome as a driver.

First install it locally to run your test-suite:

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
    chromeOptions: { args: %w[headless disable-gpu] }
  )

  Capybara::Selenium::Driver.new(app, browser: :chrome, desired_capabilities: capabilities)
end

Capybara.javascript_driver = :headless_chrome

# ...
```

That's all. Run `bin/cucumber` to check your tests.

## License

This repo content is released under the [MIT License](http://www.opensource.org/licenses/MIT).

Copyright (c) 2017 Nikita Bulai (bulajnikita@gmail.com).
