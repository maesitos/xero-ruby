source 'https://rubygems.org'

gemspec

# Pin activesupport to specific version for compatibility
gem 'activesupport', '3.2.22.5'

group :development, :test do
  gem 'rake', '~> 13.2.1'
  # pinned to support Ruby 2.6.0 (pry-byebug 3.10+ requires Ruby 2.7+)
  gem 'pry-byebug', '< 3.10.0'
  # relaxed to support Ruby 2.6.0 (rubocop 1.50+ requires Ruby 2.7+)
  gem 'rubocop', '~> 1.21'
  gem 'bundler-audit'
end
