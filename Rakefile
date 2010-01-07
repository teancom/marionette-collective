# Rakefile to build a project using HUDSON

require 'rake/rdoctask'
require 'rake/clean'

PROJ_NAME = "mcollective"
PROJ_FILES = ["build/doc", "#{PROJ_NAME}.spec", "#{PROJ_NAME}.init", "mcollectived.rb", "etc", "lib", "plugins", "COPYING", "ext"]
PROJ_FILES << Dir.glob("mc-*")
PROJ_DOC_TITLE = "The Marionette Collective"
PROJ_VERSION = "0.4.1"
PROJ_RELEASE = "1"
PROJ_RPM_NAMES = [PROJ_NAME]

ENV["RPM_VERSION"] ? CURRENT_VERSION = ENV["RPM_VERSION"] : CURRENT_VERSION = PROJ_VERSION
ENV["BUILD_NUMBER"] ? CURRENT_RELEASE = ENV["BUILD_NUMBER"] : CURRENT_RELEASE = PROJ_RELEASE

CLEAN.include("build")

def announce(msg='')
    STDERR.puts "================"
    STDERR.puts msg
    STDERR.puts "================"
end

def mkdeb(pkg='')
    FileUtils.mkdir_p("build/deb/#{PROJ_NAME}#{pkg}/DEBIAN")

    system %{cp COPYING build/deb/#{PROJ_NAME}#{pkg}/DEBIAN/copyright}
    system %{cp ext/debian/#{PROJ_NAME}#{pkg}/* build/deb/#{PROJ_NAME}#{pkg}/DEBIAN}
    system %{echo "Version: #{CURRENT_VERSION}-#{CURRENT_RELEASE}" >> build/deb/#{PROJ_NAME}#{pkg}/DEBIAN/control}

    system %{fakeroot dpkg-deb --build build/deb/#{PROJ_NAME}#{pkg} build/#{PROJ_NAME}#{pkg}-#{CURRENT_VERSION}-#{CURRENT_RELEASE}.deb}
end

def init
    FileUtils.mkdir("build") unless File.exist?("build")
end

desc "Build documentation, tar balls and rpms"
task :default => [:clean, :doc, :archive, :rpm, :tag] 

# taks for building docs
rd = Rake::RDocTask.new(:doc) { |rdoc|
    announce "Building documentation for #{CURRENT_VERSION}"

    rdoc.rdoc_dir = 'build/doc'
    rdoc.template = 'html'
    rdoc.title    = "#{PROJ_DOC_TITLE} version #{CURRENT_VERSION}"
    rdoc.options << '--line-numbers' << '--inline-source' << '--main' << 'MCollective'
}

desc "Create a tarball for this release"
task :archive => [:clean, :doc] do
    announce "Creating #{PROJ_NAME}-#{CURRENT_VERSION}.tgz"

    FileUtils.mkdir_p("build/#{PROJ_NAME}-#{CURRENT_VERSION}")
    system("cp -R #{PROJ_FILES.join(' ')} build/#{PROJ_NAME}-#{CURRENT_VERSION}")
    system("cd build && /bin/tar --exclude .svn -cvzf #{PROJ_NAME}-#{CURRENT_VERSION}.tgz #{PROJ_NAME}-#{CURRENT_VERSION}")
end

desc "Tag the release in SVN"
task :tag => [:rpm] do
    ENV["TAGS_URL"] ? TAGS_URL = ENV["TAGS_URL"] : TAGS_URL = `/usr/bin/svn info|/bin/awk '/Repository Root/ {print $3}'`.chomp + "/tags"

    raise("Need to specify a SVN url for tags using the TAGS_URL environment variable") unless TAGS_URL

    announce "Tagging the release for version #{CURRENT_VERSION}-#{CURRENT_RELEASE}"
    system %{svn copy -m 'Hudson adding release tag #{CURRENT_VERSION}-#{CURRENT_RELEASE}' ../#{PROJ_NAME}/ #{TAGS_URL}/#{PROJ_NAME}-#{CURRENT_VERSION}-#{CURRENT_RELEASE}}
end

desc "Creates a RPM"
task :rpm => [:archive] do
    announce("Building RPM for #{PROJ_NAME}-#{CURRENT_VERSION}-#{CURRENT_RELEASE}")

    sourcedir = `rpm --eval '%_sourcedir'`.chomp
    specsdir = `rpm --eval '%_specdir'`.chomp
    srpmsdir = `rpm --eval '%_srcrpmdir'`.chomp
    rpmdir = `rpm --eval '%_rpmdir'`.chomp
    lsbdistrel = `lsb_release -r -s | cut -d . -f1`.chomp
    lsbdistro = `lsb_release -i -s`.chomp

    case lsbdistro
        when 'CentOS'
            rpmdist = ".el#{lsbdistrel}"
        else
            rpmdist = ""
    end

    system %{cp build/#{PROJ_NAME}-#{CURRENT_VERSION}.tgz #{sourcedir}}
    system %{cp #{PROJ_NAME}.spec #{specsdir}}

    system %{cd #{specsdir} && rpmbuild -D 'version #{CURRENT_VERSION}' -D 'rpm_release #{CURRENT_RELEASE}' -D 'dist #{rpmdist}' -ba #{PROJ_NAME}.spec}

    system %{cp #{srpmsdir}/#{PROJ_NAME}-#{CURRENT_VERSION}-#{CURRENT_RELEASE}#{rpmdist}.src.rpm build/}

    system %{cp #{rpmdir}/*/#{PROJ_NAME}*-#{CURRENT_VERSION}-#{CURRENT_RELEASE}#{rpmdist}.*.rpm build/}
end

desc "Create the .debs"
task :deb => [:archive] do
    announce("Building debian packages")

    FileUtils.mkdir_p("build/deb")
    Dir.chdir("build/deb") do
        system %{tar -xzf ../#{PROJ_NAME}-#{CURRENT_VERSION}.tgz}
        system %{cp ../#{PROJ_NAME}-#{CURRENT_VERSION}.tgz #{PROJ_NAME}_#{CURRENT_VERSION}.orig.tar.gz}

        Dir.chdir("#{PROJ_NAME}-#{CURRENT_VERSION}") do
            system %{cp -R ext/debian .}
            system %{debuild -i -us -uc -b}
        end

        system %{cp *.deb ..}
    end

end

# vi:tabstop=4:expandtab:ai
