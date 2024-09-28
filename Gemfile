# frozen_string_literal: true

source "https://rubygems.org"

# Jekyll theme
gem "jekyll-theme-chirpy", "~> 7.1", ">= 7.1.1"

# HTML Proofer for testing (only in test environment)
gem "html-proofer", "~> 5.0", group: :test

# Platform-specific gems for Windows
platforms :mingw, :x64_mingw, :mswin, :jruby do
  gem "tzinfo", ">= 1", "< 3"
  gem "tzinfo-data"
end

# WDM for Windows file monitoring, but exclude it for Ruby >= 3.2.0
if Gem.win_platform? && RUBY_VERSION < '3.2.0'
  gem "wdm", "~> 0.1.1", :platforms => [:mingw, :x64_mingw, :mswin]
end
