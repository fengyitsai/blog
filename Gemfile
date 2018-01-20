source 'https://rubygems.org'

require 'json'
require 'open-uri'
versions = JSON.parse(open('https://pages.github.com/versions.json').read)

gem 'github-pages', versions['github-pages']
gem 'rake'
gem 'jekyll', versions['jekyll']
gem 'jekyll-paginate', versions['jekyll-paginate']
gem 'rouge', versions['rouge']