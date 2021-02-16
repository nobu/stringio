require "bundler/gem_tasks"
require "rake/testtask"

name = "stringio"

Rake::TestTask.new(:test) do |t|
  ENV["RUBYOPT"] = "-Ilib"
  t.libs << "test" << "test/lib"
  t.ruby_opts << "-rhelper"
  t.test_files = FileList["test/**/test_*.rb"]
end

task :sync_tool do
  require 'fileutils'
  FileUtils.cp "../ruby/tool/lib/test/unit/core_assertions.rb", "./test/lib"
  FileUtils.cp "../ruby/tool/lib/envutil.rb", "./test/lib"
  FileUtils.cp "../ruby/tool/lib/find_executable.rb", "./test/lib"
end

require 'rake/extensiontask'
Rake::ExtensionTask.new(name)

task :default => [:compile, :test]

task "build" => "date_epoch"
task "date_epoch" do
  ENV["SOURCE_DATE_EPOCH"] = IO.popen(%W[git -C #{__dir__} log -1 --format=%ct], &:read).chomp
end

helper = Bundler::GemHelper.instance
def helper.version=(v)
  gemspec.version = v
  tag_version
end

def helper.tag_version
  v = version.to_s
  src = "ext/stringio/stringio.c"
  File.open(File.join(__dir__, src), "r+b") do |f|
    code = f.read
    code.sub!(/^#define\s+STRINGIO_VERSION\s+\K".*"/) {v.dump}
    f.rewind
    f.write(code)
    f.truncate(f.pos)
  end
  # system("git", "--no-pager", "-C", __dir__, "diff", "-U0", src, exception: true)
  system("git", "-C", __dir__, "commit", "-mBump version to #{version}", src, exception: true)
  super
end

major, minor, teeny = helper.gemspec.version.segments

task "bump:teeny" do
  helper.version = Gem::Version.new("#{major}.#{minor}.#{teeny+1}")
end

task "bump:minor" do
  helper.version = Gem::Version.new("#{major}.#{minor+1}.0")
end

task "bump:major" do
  helper.version = Gem::Version.new("#{major+1}.0.0")
end

task "bump" => "bump:teeny"
