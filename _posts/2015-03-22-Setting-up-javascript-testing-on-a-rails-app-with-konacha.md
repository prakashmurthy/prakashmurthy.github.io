---
layout: post
title:  "Setting up javascript testing on a rails app with konacha"
date:   2015-03-22 21:18:41
categories: 
---

Over the last year or so, the one thing that has contributed significantly to make me a better rails developer has been the deliberate practice of learning how to write model & controller tests in rails. Being able to write tests (either model level or controller level) upfront for the required feature has significantly improved the velocity with which I crank out new features on a system. And having a basic set of tests help in making changes quickly when a feature details are further clarified. The [Rails Guides chapter on Testing](http://guides.rubyonrails.org/testing.html) has been an invaluable reference in this context.

However, I have not been applying the same rigor in testing to the javascript functionality I implement. So I have been trying to address this gap in my skillset and have made a good start by being able to setup javascript testing on a rails app with konacha. This blogpost is for documenting the steps I followed for the task.

# Why Konacha?
There are numerous options out there for testing javascript, and I have found it to be an intimidating task to pick one option before. I came across [konacha](https://github.com/jfirebaugh/konacha) in an [Upcase video on unit testing javascript](https://upcase.com/videos/unit-testing-javascript). Decided to go with it thinking `If it is good for Thoughtbot, it is good for me!`.

# Getting started with Konacha - a useful resource
The above mentioned video itself wasn't too helpful in getting familiar with konacha, partly because I was encountering all the concepts for the first time, and partly because the video dealt with coffeescript + rspec while I was looking for a javascript + testunit option. 

Googling led me to an excellent blogpost by [Jo Liss](https://twitter.com/jo_liss) titled [Getting Started With Konacha: JavaScript Testing on Rails](http://www.solitr.com/blog/2012/04/konacha-tutorial-javascript-testing-with-rails/) that was exactly what I was looking for. This blogpost gives a simple example of unit testing a plain javascript function, detailing the steps along the way. 

# Steps following the above mentioned blogpost:
The steps I followed were the exact same as the ones Jo Liss mentions in her blogpost, with one exception; I will repeat the complete steps here, highlighting the differences.

1) Include `konacha` gem in the Gemfile:

```
group :development, :test do
  gem 'konacha'
end
```

2) Create the `test/javascripts` folder, and create a test file:

Followed the details in the blogpost, except that I used `./test` folder instead of `./spec` folder in this step as I wanted to work with `testunit` and not `rspec`.

3) Customize konacha to work with `test/` instead of `spec`:

The details for this step are found in the [konacha configuration documentation](https://github.com/jfirebaugh/konacha#configuration). Following the advice there, I added a `config/initializers/konacha.rb` file with the following customization:

```
Konacha.configure do |config|
  config.spec_dir     = "test/javascripts"
  config.spec_matcher = /_test\./
  config.stylesheets  = %w(application)
  config.driver       = :selenium
end if defined?(Konacha)
```

4) Run `bundle exec rake konacha:serve` and point the browser at [localhost:3500](http://localhost:3500)

**Voila!** The tests are run, and the results are shown in the browser.

![Test resuls in the browser]({{ site.url }}/images/test_output_browser.png)

# Running the tests from command-line:

So far so good. I have been able to setup a testing framework in place, run a javascript unit test from the browser. However, my final goal is to be able to have my Continuous Integration server ([circleci](https://circleci.com)) run these tests; so I have to figure out how to run these tests from the command-line.

Turns out it is an easy thing to do!


1) Run `rake konacha:run` for the tests to run from the command line.

This step requires `selenium webdriver` to be included; so added the following to the `Gemfile`:

```
group :test do
  gem 'selenium-webdriver'
end
```

Now when I run `rake konacha:run` step from the command line, I get the following output:

```
$ rake konacha:run
...

Finished in 0.00 seconds
3 examples, 0 failed, 0 pending
```

**Yay!**

# Making `rake test` run the javascript tests as well:

Running `rake konacha:run` makes the tests run from the command-line. The CI server runs `rake test` to run all the tests. So I do I make `rake test` run the javascript tests as well? 

Found the answer for this question from a blogpost by [Ken Collins](https://twitter.com/metaskills) titled [Customizing Rake Tasks In Rails 4.1 And Higher](http://metaskills.net/2015/02/08/customizing-rake-tasks-in-rails-41-and-higher/)

Based on Ken's advice, I added the following to `Rakefile` on my app:

```
Rake::Task['test:run'].clear

namespace :test do
  task 'js' => ['konacha:run']

  Rake::TestTask.new(:_run) do |t|
    t.libs << "test"
    t.test_files = FileList['test/**/*_test.rb']
  end
  task 'run' => ['test:js', 'test:_run']
end
```
Please do read the blogpost to understand what these steps do.

Now running `rake test:js` runs the javascript tests; and running `rake test` runs all the tests, including the javascript tests!

# One last thing to handle:
`rake test` did not succeed on the first attempt though, instead giving a long error trace with the following text included:

```
rake aborted!
WebMock::NetConnectNotAllowedError: Real HTTP connections are disabled. Unregist
ered request: GET http://127.0.0.1:61266/__identify__ with headers {'Accept'=>'*
/*', 'Accept-Encoding'=>'gzip;q=1.0,deflate;q=0.6,identity;q=0.3', 'User-Agent'=
>'Ruby'}

You can stub this request with the following snippet:

stub_request(:get, "http://127.0.0.1:61266/__identify__").with(:headers => 
{'Accept'=>'*/*', 'Accept-Encoding'=>'gzip;q=1.0,deflate;q=0.6,identity;q=0.3', 
'User-Agent'=>'Ruby'}).to_return(:status => 200, :body => "", :headers => {})
```

This is because I use `webmock` on this app, and that prevents all outbound HTTP requests including the http call made to selenium webdriver. 

[Stack Overflow to the rescue here!](http://stackoverflow.com/a/11668778/429758) Added the following to the `test/environments/test.rb` file:

```
Rails.application.configure do
  ...
  WebMock.disable_net_connect!(:allow_localhost => true)
end
```

That did the trick, and `rake test` worked perfectly fine on my local machine. 

# Getting CI server to run all the tests, including javascript tests. 

To my very pleasant surprise, I did not have to do anything else. Pushed the code up to github, and [circleci](https://circleci.com) ran all the tests successfully!

![Circle CI test execution]({{ site.url }}/images/circleci_test.png)

# Next steps:
Now that the rails app is setup for testing the javascript code, I want to improve my javascript unit testing skills, and see the following as the next steps:

1. Get familiar with [mocha test framework](http://mochajs.org) and [chai assertion library](http://chaijs.com); these are used internally by konacha.
2. Write tests for some of the existing javascript code, changing the code to make it testable along the way.
3. Hopefully in a couple of months time, be able to test-drive javascript code in rails.

![konacha]({{ site.url }}/images/konacha.jpg)
