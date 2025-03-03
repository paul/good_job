# frozen_string_literal: true

begin
  require 'bundler/setup'
rescue LoadError
  puts 'You must `gem install bundler` and `bundle install` to run rake tasks'
end

require 'dotenv/load'
require_relative "lib/good_job/version"
Rake.load_rakefile 'spec/test_app/Rakefile'
Bundler::GemHelper.install_tasks

require 'rdoc/task'

RDoc::Task.new(:rdoc) do |rdoc|
  rdoc.rdoc_dir = 'rdoc'
  rdoc.title    = 'GoodJob'
  rdoc.options << '--line-numbers'
  rdoc.rdoc_files.include('README.md')
  rdoc.rdoc_files.include('lib/**/*.rb')
end

def system!(*args)
  system(*args) || abort("\n== Command #{args} failed ==")
end

desc 'Commit version and changelog'
task :release_good_job, [:version_bump] do |_t, args|
  version_bump = args[:version_bump]
  if version_bump.nil?
    puts "Pass a version [major|minor|patch|pre|release] or a given version number [x.x.x]:"
    puts "$ bundle exec rake release[VERSION_BUMP]"
    exit(1)
  end

  puts "\n== Bumping version number =="
  system! "gem bump --no-commit --version #{version_bump}"

  puts "\n== Reloading GoodJob::VERSION"
  load File.expand_path('lib/good_job/version.rb', __dir__)
  puts GoodJob::VERSION

  puts "\n== Updating Changelog =="
  system! ENV, "bundle exec github_changelog_generator --user bensheldon --project good_job --future-release v#{GoodJob::VERSION}"

  puts "\n== Updating Gemfile.lock version =="
  system! "bundle update --conservative good_job"

  puts "\n== Verifying Gemfile.lock =="
  gemfile_lock = File.read(File.join(File.dirname(__FILE__), 'Gemfile.lock'))
  unless gemfile_lock.include?("jdbc-postgres")
    puts "ABORTING...\nMissing JRuby library. Gemfile.lock has possibly been corrupted. Inspect the diff."
    exit(1)
  end

  puts "\n== Creating gem package and sha512 checksum =="
  system! "bundle exec rake build:checksum"

  puts "\n== Creating sha256 checksum too =="
  require "digest/sha2"
  gem_filename = "good_job-#{GoodJob::VERSION}.gem"
  sha256_checksum = Digest::SHA256.hexdigest File.read "#{__dir__}/pkg/#{gem_filename}"
  File.write "#{__dir__}/checksums/#{gem_filename}.sha256", "#{sha256_checksum}\n"

  puts "\n== Creating git commit  =="
  system! "git add lib/good_job/version.rb CHANGELOG.md Gemfile.lock checksums"
  system! "git commit -m \"Release good_job v#{GoodJob::VERSION}\""
  system! "git tag v#{GoodJob::VERSION}"

  require 'cgi'
  changelog_anchor = "v#{GoodJob::VERSION.delete('.')}-#{Time.now.utc.strftime('%Y-%m-%d')}"
  changelog_url = "https://github.com/bensheldon/good_job/blob/main/CHANGELOG.md##{changelog_anchor}"

  puts "\n"
  puts <<~INSTRUCTIONS
    == Next steps ==

    Run the following commands:

    1. Push commit and tag to Github:
        git push origin && git push origin --tags
    2. Push to Rubygems.org:
        gem push pkg/good_job-#{GoodJob::VERSION}.gem
    3. Author a Github Release:
        https://github.com/bensheldon/good_job/releases/new?tag=v#{GoodJob::VERSION}&body=#{CGI.escape "_Review the [Changelog](#{changelog_url}) for more details._"}
  INSTRUCTIONS
end

require 'rspec/core/rake_task'
RSpec::Core::RakeTask.new(:rspec)

task default: :rspec
