require 'rake/clean'
require 'rdf_context'
require 'net/http'
require 'mustache'
require 'time'
require 'git'

SPDX_SPEC_FILE_NAME = "build/spdx-DRAFT.html"
SPDX_ONT_FILE_NAME = 'build/spdx-ont-DRAFT.rdf'
SPDX_TVG_FILE_NAME = 'build/spdx-grammar-DRAFT.txt'
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
      repo = Git.open(File.dirname(__FILE__))
      "DRAFT (#{Time.now.utc.strftime("%d %b %Y %H:%M %Z")} - #{repo.current_branch} #{repo.gtree('HEAD').sha[0,6]})"
    end
  end

  File.open(t.name, 'w'){|f| f.write compiler.render}
end

task :default => [SPDX_SPEC_FILE_NAME, SPDX_ONT_FILE_NAME, SPDX_TVG_FILE_NAME]
