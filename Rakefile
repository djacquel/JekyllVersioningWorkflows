# -------------------------------------------------------
# Jekyll on Github page complex workflow
# copyleft djacquel 2014
# -------------------------------------------------------
# This rake file helps the setup of a Jekyll install for complex workflow.
#
#
# - Tasks involving Jekyll plugins or tasks (grunt, ...) that are not
# supported by Github pages.
# -
# -------------------------------------------------------
#
# ## Use
#
# ### Initial setup
#
# In order to setup you worflow :
#
# - create a Github repository
# - Copy this file in your working directory as *Rakefile*
# - set parameters below
# - In a console do *cd pathTo/yourWorkingDirectory/*
# - then launch the install with *rake setup*
#
# ### Deploying
#
# When your happy with your code you can deploy with :
#
#    rake deploy
#
# This will :
#   - build your site with production setup
#   - push to code and site branches
#
#
# ## Troubleshooting
#
# If you cannot troubleshoot by yourself, open an issue on Github https://github.com/djacquel/JekyllVersioningWorkflows/issues
#
#
#    CONTRIBUTORS ARE WELCOME

require 'fileutils'
require 'yaml'
require 'tempfile'

###############################################################
#       EDIT THIS CONFIGURATION PARAMETERS                    #
###############################################################

# UO refers to User / Organization repository
# P  refers to Project repository

# your github user name
GithubUserName   = 'djacquel'

# Github target repository
# you must be allowed to contribute to this repository
GithubRepository = 'JekyllVersioningWorkflows'

# User / Organization sources branch
UODefaultSourceBranch = 'sources'

# Project repository sources branch
PDefaultSourceBranch = 'master'

# repository remote name
GithubRepositoryRemoteName = 'origin'

# configuration root key used by this rake file
# you can change it if you have collision with an existing key
# in you _config.yml file
ConfigGithubKey = 'gh-pages'

# where do we store site files
# this folder name MUST be prepended with an underscore
# in order to be ignored by Git
# change this only if you really need
SiteFileFolder = '_site'


# debug - if set to true, all system call are replaced by puts
DoDebug = false

###############################################################
#       DO NOT CHANGE PARAMETERS BELOW THIS LINE !!!         #
###############################################################

UOSiteBranch = 'master'  # CANNOT be changed - DO NOT CHANGE THIS !
PSiteBranch  = 'gh-pages'# CANNOT be changed - DO NOT CHANGE THIS !

ConfigFile   = '_config.yml'

task :default => :dev

desc "Do the base setup for a repository with complex workflow"
task :setup do |t|

  init

  # generates base Jekyll sources
  jekyllnew

  # write Jekyll configuration file
  writeconfig

  # create a .nojekyll file in order to instruct Github not to proccess
  system_call( 'touch .nojekyll') )

  # build site for first time
  Rake::Task[:build].invoke

  puts 'INTERUPT !!!!!'
  exit(999)


  # connect local files with Github repository
  connect_sources
  connect_site

  puts"Workflow setup OK"
end

desc "Build Jekyll with production configuration"
task :build do |t|
  puts "Building with production parameters"
  system_call('jekyll build')
end

desc "Deploy code and site in appropriate branches"
task :deploy do |t|

  init

  puts "deploying to " + GithubUserName + "/" + GithubRepository

  Rake::Task[:build].invoke

  # push the code
  git_add_and_commit
  git_push

  # push the site
  Dir.chdir(SiteFileFolder) do
    git_add_and_commit
    git_push
  end

  puts "\n## Github Pages deploy complete"
end

desc "Build Jekyll with development configuration"
task :dev do |t|
  puts "Building with dev parameters"
  sh 'jekyll build --config _config.yml,_config_dev.yml --trace'
end

desc "Build Jekyll with development configuration and watch"
task :w do |t|
  puts "Building with dev parameters and watch"
  sh 'jekyll build --config _config.yml,_config_dev.yml -w'
end

desc "list tasks"
task :list do
  puts "Tasks: #{(Rake::Task.tasks - [Rake::Task[:list]]).join(', ')}"
  puts "(type rake -T for more detail)\n\n"
end

# init some vars
def init
  @repositpryGitPath = "git@github.com:" + GithubUserName + "/" + GithubRepository + ".git"
  @siteBaseUrl       = "http://" + GithubUserName + ".github.io"
  # determine if it's UO or P repository
  userRepo = GithubUserName + ".github.io"

  if GithubRepository == userRepo
    puts "#{GithubRepository} is a User / Organization repository"
    @siteUrl      = @siteBaseUrl
    @sourceBranch = UODefaultSourceBranch
    @siteBranch   = UOSiteBranch
    @baseurl      = ""
  else
    puts "#{GithubRepository} is a Project repository"
    @siteUrl      = @siteBaseUrl + "/" + GithubRepository
    @sourceBranch = PDefaultSourceBranch
    @siteBranch   = PSiteBranch
    @baseurl      = "/" + GithubRepository
  end

  puts "Source branch will be #{@sourceBranch}"
  puts "Site branch will be #{@siteBranch}"
end

# Create Jekyll sources in root folder"
def jekyllnew

  @curPath = File.dirname(__FILE__)

  # ASK USER BEFORE REMOVING FILES
  if ask("\nThis will remove ALL your files and create a new Jekyll site in #{@curPath}. \nContinue ?", ['yes','no']) == 'yes'
    puts "Ok, lets go"
  else
    puts'Exiting rake task'
    exit(999)
  end

  # remove old files except this Rakefile
  Dir['*', '\.git*', "\.sass*"].reject{ |f| f['Rakefile'] }.each do |filename|
    # be sure that we work in the path
    filename = sanitized_path(@curPath, filename)
    puts "removing old files : #{filename}"
    rm_rf filename
  end

  # jekyll new in root folder
  system_call( "jekyll new . --force", false )

  # verify that git will ignore destination folder
  # because this folder is versioned by itself
  check_gitignore(SiteFileFolder)

  # ignore various files and folder
  check_gitignore(".sass-cache")

end

# Write Jekyll config file
def writeconfig
  # useless
  # config_add_key('sources_branch', @sourceBranch)
  # config_add_key('site_branch', @siteBranch)
  config_add_key('url', @siteUrl, true)
  config_add_key('baseurl', @baseurl, true)
  config_add_key('destination', SiteFileFolder, true)

  config_write
end

# set a configuration key
# @param string key
# @param mixed value
# @param boolean atRootLevel -
#       if true, key: value is set at root level
#       if false, key: value is set in ConfigGithubKey
def config_add_key(key, value, atRootLevel = false)

  puts "Add config key #{key} : #{value}"
  config_get

  if atRootLevel
    @config[key] = "#{value}"
  else
    if !@config.has_key? ConfigGithubKey
      @config[ConfigGithubKey] = {}
    end

    @config[ConfigGithubKey][key] = "#{value}"
  end

end

# write _config.yml file
def config_write()
  puts "Writing config"
  File.open(ConfigFile, 'w') { |f| YAML.dump(@config, f) }
  config_refresh
end

# get config from _config.yml file
def config_get
  @config    ||= YAML.load_file(ConfigFile)
end

# refresh config variable
def config_refresh
  @config = nil
  config_get
end

# can be factorized with connect_site
# connect sources files with Github repository
def connect_sources
  git_init_and_remote_add
  git_branch(@sourceBranch)
  git_add_and_commit
  git_push
end

# connect site files with Github repository
def connect_site
  Dir.chdir(SiteFileFolder) do
    git_init_and_remote_add
    git_branch(@siteBranch)
    git_add_and_commit
    git_push
  end
end

# be sure that .gitignore will ignore *destination* folder
# we add an entry if it doesn't exist
def check_gitignore(fileName)
  @fileIgnored = false
  t_file = Tempfile.new('gitignore.temp')

  File.open(".gitignore", 'r') do |f|
    f.each_line do |l|
      t_file.puts l
      if l == fileName
        @fileIgnored = true
      end
    end

    if @fileIgnored == false
      t_file.puts fileName
    end

  end
  t_file.close
  FileUtils.mv(t_file.path, ".gitignore")

  puts"set #{fileName} to be ignored by git"
end

def git_init_and_remote_add
  system_call( "git init", false )
  system_call( "git remote add #{GithubRepositoryRemoteName} #{@repositpryGitPath}", false )
end

def git_branch(branch)
  system_call( "git checkout -b #{branch}", false )
  get_current_branch
end

def git_add_and_commit
  system_call( "git add -A", false )
  @defaultMessage ||= "Code update #{Time.now.utc}"
  get_current_branch
  @message = ask("Commit message (default = #{@defaultMessage}) : ")
  if @message == ''
    @message = @defaultMessage
  else
    @defaultMessage = @message
  end

  @message = @current_branch.strip + " branch - " + @message.strip

  system_call( "git commit -m '#{@message}'", false )
end

def git_push
  system_call( "git push #{GithubRepositoryRemoteName} #{@current_branch}", false )
end

def get_current_branch
  @current_branch = `git symbolic-ref --short HEAD`
  @current_branch.strip
  puts"current branch is #{@current_branch}"
end

def system_call(command, debug = true)

  puts "\n------------------ System : #{command}"

  if DoDebug == false || debug == false
    system command
  end
end

def get_stdin(message)
  print message
  STDIN.gets.chomp
end

def ask(message, valid_options = [])

  if valid_options.length > 0
    answer = get_stdin("#{message} #{valid_options.to_s.gsub(/"/, '').gsub(/, /,'/')} ") while !valid_options.include?(answer)
  else
    answer = get_stdin(message)
  end
  answer
end


# from Jekyll
# Ensures the questionable path is prefixed with the base directory
# and prepends the questionable path with the base directory if false.
#
# base_directory - the directory with which to prefix the questionable path
# questionable_path - the path we're unsure about, and want prefixed
#
# Returns the sanitized path.
def sanitized_path(base_directory, questionable_path)
  return base_directory if base_directory.eql?(questionable_path)

  clean_path = File.expand_path(questionable_path, "/")
  clean_path = clean_path.sub(/\A\w\:\//, '/')

  unless clean_path.start_with?(base_directory.sub(/\A\w\:\//, '/'))
    File.join(base_directory, clean_path)
  else
    clean_path
  end
end
