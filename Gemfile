source "https://rubygems.org"

# Jekyll 버전
gem "jekyll", "~> 4.3.3"

# 기본 테마 (원하는 테마로 변경 가능)
gem "minima", "~> 2.5"

# Jekyll 플러그인
group :jekyll_plugins do
  gem "jekyll-feed", "~> 0.12"
  gem "jekyll-gist"  # GitHub Gist를 삽입하는 플러그인
  gem 'faraday-retry'

end

# Windows에서 필요할 경우 사용하는 gem
platforms :mingw, :x64_mingw, :mswin, :jruby do
  gem "tzinfo", ">= 1", "< 3"
  gem "tzinfo-data"
end

# Windows에서 디렉토리 감시 성능 향상
gem "wdm", "~> 0.1.1", :platforms => [:mingw, :x64_mingw, :mswin]

# JRuby 환경에서 특정 gem을 고정
gem "http_parser.rb", "~> 0.6.0", :platforms => [:jruby]
