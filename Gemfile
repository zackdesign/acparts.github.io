source "https://rubygems.org"

# Hello! This is where you manage which Jekyll version is used to run.
# When you want to use a different version, change it below, save the
# file and run `bundle install`. Run Jekyll with `bundle exec`, like so:
#
#     bundle exec jekyll serve

gem "jekyll", "~> 4.3"

# Ruby 3.4+ dropped these from the default gems; Jekyll still relies on them.
gem "csv"
gem "base64"
gem "bigdecimal"

# If you have any plugins, put them here!
group :jekyll_plugins do
  gem 'jekyll-seo-tag'
end

# Contentful import — used only by `bundle exec rake contentful:import` to
# refresh _data/contentful/spaces/acparts.yaml. Not needed to build the site
# (the generated data is committed). Replaces the archived
# jekyll-contentful-data-import gem with the maintained official SDK.
gem "contentful", "~> 2.20"
gem "rake"
