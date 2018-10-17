---
layout: epic
title: "Switch from Capybara Webkit to Chrome"
date: 2018-10-16
author: chungyi
categories: [testing, ci, rspec, rails]
---

Volt is the internal app name of Artsy CMS, and our partners use it to manage their inventory and presence on artsy.net. It's a Rails-based UI app that talks to many API services. We use [RSpec][rspec] extensively to cover controller, model, view, and feature specs. As of Jun. 2018, Volt had 3751 specs and 495 of them were run with JavaScript enabled. It took about 18 mins to run on CircleCI with 6x parallelism.

Capybara-webkit was introduced from the very beginning of Volt for testing JavaScript-enabled specs via headless WebKit browser. It's been providing a lot of confidence for the past 4+ years; however, a few reasons/growing concerns have encouraged us to look for (more modern) alternatives:

<!-- more -->

## The Problem

- The dependency of specific versions of Qt has been causing frustrations to set it up properly both on devs' local machines and on CI.
- The roadmap of future development of capybara-webkit is unclear.
- It's been hard to truly identify the root cause of flickering specs, while retrying tended to resolve it on CI.
- The entire Volt tests took about 20 mins to complete on CI, with 6 parallelism. The speed of JavaScript specs made it unrealistic to run locally.

## The Goal

Headless Chrome has gained a lot of attention in the past years and migrations done by companies like [GitLab][headless-chrome-migration-gitlab] and [thoughtbot][headless-chrome-migration-thoughtbot] have proved it to be a promising alternative to capybara-webkit. In fact, it's been [officially included in Rails 5.1][rails-5.1-system-tests] for system tests.

The goal of this project is to switch to Headless Chrome and maintain the same feature sets we have now. This includes:

- Be able to make all existing specs pass
- Be able to run in container environment like our Hokusai test
- Be able to examine browser console logs
- Be able to take screenshots on demand and automatically when failed
- Be able to get/set cookies
- And more details in each commit

## The How
_(Insert instructions and code snippet)_

https://github.com/artsy/volt/pull/3064

## Lessons learned

Similar to the previous attempt, switching to Headless Chrome caused about 60 spec failures on my local, and it seemed to still include flickering ones.

I basically just went through them one by one and fixed them. A big part of the spec code changes was to migrate to Selenium WebDriver's way of achieving things that were previously done via [capybara-webkit's non-standard driver methods][capybara-webkit-non-standard-driver-methods]. I grouped similar fixes into the same commits and documented them. It may be easier to follow by going through individual commits.

I was able to identify one possible cause of flickering issue and resolve it in https://github.com/artsy/volt/pull/3064/commits/96bf6dccca714c3a06dfcc13b91d7944de10950b. It seemed to have more, and _I think_ some might be related to how we wrote specs that didn't play perfectly with async behavior.

Regarding speed, I didn't see improvement after switching to Headless Chrome, as also mentioned in GitLab's blog post and others.

## Next steps

- Update Volt to Rails 5.1 and switch to system tests
- Keep migrating false view specs to feature specs/system tests
- Keep investigating flickering specs
- Experiment on speed improvement
- Experiment using a non-root user in Docker image to get rid of the `--no-sandbox` Chrome arg
- Look into suspicious spec failures on CI (that got fixed after retrying)

[rspec]: https://github.com/rspec/rspec
[headless-chrome-migration-gitlab]: https://about.gitlab.com/2017/12/19/moving-to-headless-chrome/
[headless-chrome-migration-thoughtbot]: https://robots.thoughtbot.com/headless-feature-specs-with-chrome
[capybara-webkit-non-standard-driver-methods]: https://github.com/thoughtbot/capybara-webkit#non-standard-driver-methods
[rails-5.1-system-tests]: http://guides.rubyonrails.org/5_1_release_notes.html#system-tests
