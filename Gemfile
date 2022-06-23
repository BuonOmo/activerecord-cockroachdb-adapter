require 'openssl'
source 'https://rubygems.org'
gemspec

if ENV['RAILS_SOURCE']
  gemspec path: ENV['RAILS_SOURCE']
else
  def get_version_from_gemspec
    gemspec = eval(File.read('activerecord-cockroachdb-adapter.gemspec'))

    gem_version = gemspec.dependencies.
      find { |dep| dep.name == 'activerecord' }.
      requirement.
      requirements.
      first.
      last

    major, minor, tiny, pre = gem_version.segments

    if pre
      gem_version.to_s
    else
      find_latest_matching_version(major, minor)
    end
  end

  def find_latest_matching_version(gemspec_major, gemspec_minor)
    all_activerecord_versions.
      reject { |version| version["prerelease"] }.
      map { |version| version["number"].split(".").map(&:to_i) }.
      find { |major, minor|
        major == gemspec_major && (minor == gemspec_minor || gemspec_minor.nil?)
      }.join(".")
  end

  def all_activerecord_versions
    require 'net/http'
    require 'yaml'

    uri = URI.parse "https://rubygems.org/api/v1/versions/activerecord.yaml"
    http = Net::HTTP.new(uri.host, uri.port)
    http.use_ssl = true
    http.verify_mode = OpenSSL::SSL::VERIFY_NONE

    YAML.load(
      http.request(Net::HTTP::Get.new(uri.request_uri)).body
    )
  end

  # Get Rails from source beacause the gem doesn't include tests
  version = ENV['RAILS_VERSION'] || get_version_from_gemspec
  gem 'rails', git: "https://github.com/rails/rails.git", tag: "v#{version}"
end

group :development do
  gem "byebug"
  gem "minitest-excludes", "~> 2.0.1"

  # Gems used by the ActiveRecord test suite
  gem "bcrypt", "~> 3.1.18"
  gem "mocha", "~> 1.14.0"
  gem "sqlite3", "~> 1.4.4"

  gem "minitest", "~> 5.15.0"
end
