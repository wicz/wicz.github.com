# encoding: utf-8
require "jekyll"

desc "Publish blog"
task :publish do
  Jekyll::Site.new(Jekyll.configuration({
    source:       ".",
    destination:  "_site",
    config:       "_config.yml"
  })).process

  origin = `git config --get remote.origin.url`

  Dir.mktmpdir do |tmp|
    cp_r "_site/.", tmp
    Dir.chdir tmp
    system "git init"
    system "git add . && git commit -m 'Site updated at #{Time.now.utc}'"
    system "git remote add origin #{origin}"
    system "git push origin master --force"
  end
end

task default: :publish

