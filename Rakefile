require 'rake/clean'
require 'rdf_context'
require 'net/http'
require 'mustache'
require 'time'
require 'git'

Struct.new("SpecVersion", :version, :timestamp, :branch, :commit)
repo = Git.open(File.dirname(__FILE__))
VERSION = Struct::SpecVersion.new("DRAFT", Time.now.utc, repo.current_branch, repo.gtree('HEAD').sha)
class << VERSION
  def human_str
    "#{version} (#{timestamp.strftime("%d %b %Y %H:%M %Z")} #{branch} #{commit[0,6]})"
  end
  
  def filename_str
    "#{version}_#{timestamp.strftime("%d%b%Y%H%M")}_#{branch}_#{commit[0,6]}"
  end
end

SPDX_SPEC_FILE_NAME = "build/spdx-DRAFT.html"
SPDX_ONT_FILE_NAME = 'build/spdx-ont-DRAFT.rdf'
SPDX_TVG_FILE_NAME = 'build/spdx-grammar-DRAFT.txt'
SPDX_SCREEN_CSS = 'build/screen.css'
BUILD_DIR = 'build'

CLEAN.include BUILD_DIR

directory BUILD_DIR

desc "Translate SPDX ontology from N3 into RDF/XML"
file SPDX_ONT_FILE_NAME => SPDX_SPEC_FILE_NAME do |t|
  graph = File.open(SPDX_SPEC_FILE_NAME){|f| RdfContext::RdfaParser.new.parse(f, 'http://spdx.org/spec')}

  File.open(t.name, 'w'){|f| graph.serialize(:format => :rdfxml, :io => f)}
end


desc "Abstract ontology produced by WonderWeb"
task "abstract" => SPDX_ONT_FILE_NAME do
  res = Net::HTTP.post_form(URI.parse('http://www.mygrid.org.uk/OWL/Validator'),
                            {'rdf' => File.read(SPDX_ONT_FILE_NAME), 'abstract' => 'yes', 'level' => 'Full'})
  doc = Nokogiri::HTML.parse(res.body)
  elem = doc.at('div.box pre')

  if elem.nil?
    STDERR.puts doc.at('//body').to_s
    raise "OWL validation seems to have failed!"
  end

  puts elem.content
end

desc "Extract tag-value format grammar"
file SPDX_TVG_FILE_NAME => [BUILD_DIR, SPDX_SPEC_FILE_NAME] do |t|
  spec= File.open(SPDX_SPEC_FILE_NAME){|f| Nokogiri::HTML.parse(f) }

  File.open(t.name, 'w'){|f| f.write spec.search("code.grammar").map(&:content).join}
end

desc "Compile complete spec from sections"
file SPDX_SPEC_FILE_NAME => [BUILD_DIR] + (FileList['*.html'] - FileList[SPDX_SPEC_FILE_NAME]) do |t|

  compiler = Class.new(Mustache) do 
    self.template_path = File.dirname(__FILE__)
    self.template_extension = 'html'
    self.template_file = './spec.html'

    def spdx_version
      VERSION.human_str
    end
  end

  File.open(t.name, 'w'){|f| f.write compiler.render}
end

file SPDX_SCREEN_CSS => "screen.css" do |t|
  copy "screen.css", t.name
end

task :pkg => :default do |t|
  pkg_name = "spdx-#{VERSION.filename_str}"
  system "ln -s build #{pkg_name}"
  system "tar czHf #{pkg_name}.tar.gz #{pkg_name}"
  File.unlink pkg_name
end


task :default => [SPDX_SPEC_FILE_NAME, SPDX_ONT_FILE_NAME, SPDX_TVG_FILE_NAME, SPDX_SCREEN_CSS]
